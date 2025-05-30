---
title: FreeBSD - Periodic ZFS Snapshots
date: 2025-05-30
linktitle: FreeBSD - Periodic ZFS Snapshots
slug: freebsd-periodic-snapshots
description: Howto setup periodic ZFS snapshots on FreeBSD.
tags:
  - FreeBSD
  - ZFS
  - "#HowTo"

authors:
- harry
---
# FreeBSD - Periodic ZFS Snapshots

## Install `zfs-periodic`

```sh
doas pkg install zfs-periodic
```

Output
```
Message from zfs-periodic-1.0.20130213:

--
In order to enable periodic snapshots you need
to add these lines to your /etc/periodic.conf

hourly_output="root"
hourly_show_success="NO"
hourly_show_info="YES"
hourly_show_badconfig="NO"
hourly_zfs_snapshot_enable="YES"
hourly_zfs_snapshot_pools="tank"
hourly_zfs_snapshot_keep=6
daily_zfs_snapshot_enable="YES"
daily_zfs_snapshot_pools="tank"
daily_zfs_snapshot_keep=7
weekly_zfs_snapshot_enable="YES"
weekly_zfs_snapshot_pools="tank"
weekly_zfs_snapshot_keep=5
monthly_zfs_snapshot_enable="YES"
monthly_zfs_snapshot_pools="tank"
monthly_zfs_snapshot_keep=2

To get hourly snapshots you also need to add
something like this to /etc/crontab:

2       *       *       *       *       root    periodic hourly
```

## Customize `/etc/periodic.conf`
```config
# Hourly options
hourly_output="root"                                    # user or /file
hourly_show_success="NO"                                # scripts returning 0
hourly_show_info="YES"                                  # scripts returning 1
hourly_show_badconfig="NO"                              # scripts returning 2

# 000.zfs-snapshot
hourly_zfs_snapshot_enable="YES"
hourly_zfs_snapshot_pools="zroot"                       # space seperated list of pools or datasets
hourly_zfs_snapshot_keep=24
hourly_zfs_snapshot_skip=""                             # space seperated list of datasets to skip

# Quarter hourly options

# 000.zfs-snapshot
quarter_hourly_zfs_snapshot_enable="YES"
quarter_hourly_zfs_snapshot_pools="zroot"
quarter_hourly_zfs_snapshot_keep=4
quarter_hourly_zfs_snapshot_skip=""

# Daily options

# 000.zfs-snapshot
daily_zfs_snapshot_enable="YES"
daily_zfs_snapshot_pools="zroot"
daily_zfs_snapshot_keep=7
daily_zfs_snapshot_skip=""


# 404.status-zfs
daily_status_zfs_enable="YES"

# Weekly options

# 000.zfs-snapshot
weekly_zfs_snapshot_enable="YES"
weekly_zfs_snapshot_pools="zroot"
weekly_zfs_snapshot_keep=5
weekly_zfs_snapshot_skip=""

# Monthly options

# 000.zfs-snapshot
monthly_zfs_snapshot_enable="YES"
monthly_zfs_snapshot_pools="zroot"
monthly_zfs_snapshot_keep=3
monthly_zfs_snapshot_skip=""

# 998.zfs-scrub
monthly_zfs_scrub_enable="YES"
monthly_zfs_scrub_pools="zroot"
```

## Customize `/etc/crontab`
```cron
# /etc/crontab - root's crontab for FreeBSD
#
#
SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin
#
#minute hour    mday    month   wday    who     command
#
# Save some entropy so that /dev/random can re-seed on boot.
*/11    *       *       *       *       operator /usr/libexec/save-entropy
#
# Rotate log files every hour, if necessary.
0       *       *       *       *       root    newsyslog
#
# Perform daily/weekly/monthly maintenance.
*/15    *       *       *       *       root    periodic quarter_hourly
1       *       *       *       *       root    periodic hourly
1       3       *       *       *       root    periodic daily
15      4       *       *       6       root    periodic weekly
30      5       1       *       *       root    periodic monthly
#
# Adjust the time zone if the CMOS clock keeps local time, as opposed to
# UTC time.  See adjkerntz(8) for details.
1,31    0-5     *       *       *       root    adjkerntz -a
```

## Enable quarter hourly snapshots
see https://github.com/ross/zfs-periodic/pull/10/commits/b37103444a0dcac58937e134cbbeb347831b967a and https://evilham.com/en/blog/2023-ZFS-replication-tools/ for more details
```sh
doas mkdir /usr/local/etc/periodic/quarter_hourly
doas cp /usr/local/etc/periodic/hourly/000.zfs-snapshot /usr/local/etc/periodic//quarter_hourly/000.zfs-snapshot
```

customize `/usr/local/etc/periodic//quarter_hourly/000.zfs-snapshot`
-> change `hourly`to `quarter_hourly`
```sh
#!/bin/sh

# If there is a global system configuration file, suck it in.
#
if [ -r /etc/defaults/periodic.conf ]
then
    . /etc/defaults/periodic.conf
    source_periodic_confs
fi

pools=$quarter_hourly_zfs_snapshot_pools
if [ -z "$pools" ]; then
    pools='tank'
fi

keep=$quarter_hourly_zfs_snapshot_keep
if [ -z "$keep" ]; then
    keep=6
fi

case "$quarter_hourly_zfs_snapshot_enable" in
    [Yy][Ee][Ss])
        . /usr/local/bin/zfs-snapshot
        do_snapshots "$pools" $keep 'quarter_hourly' "$quarter_hourly_zfs_snapshot_skip"
        ;;
    *)
        ;;
esac
```

`/usr/local/bin/zfs-snapshot`
```sh
#!/bin/sh

# take the appropriately named snapshot
create_snapshot()
{
    pool=$1

    case "$type" in
        quarter_hourly)
        now=`date +"$type-%Y-%m-%d-%H-%M"`
        ;;
        hourly)
        now=`date +"$type-%Y-%m-%d-%H"`
        ;;
        daily)
        now=`date +"$type-%Y-%m-%d"`
        ;;
        weekly)
        now=`date +"$type-%Y-%U"`
        ;;
        monthly)
        now=`date +"$type-%Y-%m"`
        ;;
        yearly)
        now=`date +"$type-%Y"`
        ;;
        *)
        echo "unknown snapshot type: $type"
        exit 1
    esac

    # enumerate datasets under this pool or dataset, skip excluded datasets if requested
    if [ -n "$skip" ]; then
            egrep="($(echo $skip | sed "s/ /|/g"))"
            datasets=$(zfs list -r -H -o name $pool | egrep -v "$egrep")
    else
            datasets=$(zfs list -r -H -o name $pool)
    fi

    # loop through datasets, do snapshots of each
    for dataset in $datasets; do
        snapshot="$dataset@$now"
        # look for an existing snapshot with this name
        if zfs list $snapshot > /dev/null 2>&1; then
            echo "     snapshot $snapshot already exists, skipping..."
        else
            echo "     taking snapshot: $snapshot"
            zfs snapshot $snapshot
        fi
    done
}

# delete the named snapshot
delete_snapshot()
{
    snapshot=$1
    echo "      destroying old snapshot, $snapshot"
    zfs destroy -r $snapshot
}

# take a type snapshot of pool, keeping keep old ones
do_pool()
{
    pool=$1
    keep=$2
    type=$3
    skip=$4

    # create the regex matching the type of snapshots we're currently working
    # on
    case "$type" in
        quarter_hourly)
        # quarter_hourly-2009-01-01-00-15
        regex="^$pool@$type-[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]-[0-9][0-9]-[0-9][0-9]$"
        ;;
        hourly)
        # hourly-2009-01-01-00
        regex="^$pool@$type-[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]-[0-9][0-9]$"
        ;;
        daily)
        # daily-2009-01-01
        regex="^$pool@$type-[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]$"
        ;;
        weekly)
        # weekly-2009-01
        regex="^$pool@$type-[0-9][0-9][0-9][0-9]-[0-9][0-9]"
        ;;
        monthly)
        # monthly-2009-01
        regex="^$pool@$type-[0-9][0-9][0-9][0-9]-[0-9][0-9]"
        ;;
        yearly)
        # yearly-2009
        regex="^$pool@$type-[0-9][0-9][0-9][0-9]"
        ;;
        *)
        echo "unknown snapshot type: $type"
        exit 1
    esac

    create_snapshot $pool $type

    # get a list of all of the snapshots of this type sorted alpha, which
    # effectively is increasing date/time
    # (using sort as zfs's sort seems to have bugs)
    snapshots=`zfs list -H -o name -t snapshot | sort | grep $regex`
    # count them
    count=`echo $snapshots | wc -w`
    if [ $count -ge 0 ]; then
        # how many items should we delete
        delete=`expr $count - $keep`
        count=0
        # walk through the snapshots, deleting them until we've trimmed deleted
        for snapshot in $snapshots; do
            if [ $count -ge $delete ]; then
                break
            fi
            delete_snapshot $snapshot
            count=`expr $count + 1`
        done
    fi
}

# take snapshots of type, for pools, keeping keep old ones,
do_snapshots()
{
    pools=$1
    keep=$2
    type=$3
    skip=$4

    echo ""
    echo "Doing zfs $type snapshots:"
    for pool in $pools; do
        do_pool $pool $keep $type "$skip"
    done
}
```

## Links
- [zfs-periodic - Repo](https://github.com/ross/zfs-periodic/)
- [https://evilham.com/en/blog/2023-ZFS-replication-tools/](https://evilham.com/en/blog/2023-ZFS-replication-tools/)
- [Pull request for quarter_hourly_snapshots]( https://github.com/ross/zfs-periodic/pull/10/commits/b37103444a0dcac58937e134cbbeb347831b967a)
