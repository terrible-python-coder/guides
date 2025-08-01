# Immich Migration
High level plan:
* Create the new dataset configuration `/mnt/tank/configs/immich1` vs `/mnt/tank/configs/immich`
* Sync the data to the new dataset, using rsync `immich -> immich1`
* Spin up Immich application, as a second instance, from TrueNAS Discover Apps. Must have a unique name and port, as it will be running con-current.

## Step 1 - Create new dataset
* Create a new dataset `/mnt/tank/configs/immich1` - Apps permissions
## Step 2 - Create new sub-datasets
* Create 2x datasets underneath
   * `/mnt/tank/configs/immich1/data`
   *  `/mnt/tank/configs/immich1/db`

## Step 3 - rsync
* Run rsync to sync the datasets, with the following commands:
1. `rsync -avhz --progress /mnt/tank/configs/immich/backups/ /mnt/tank/configs/immich1/data/backups/` 
1. `rsync -avhz --progress /mnt/tank/configs/immich/library/ /mnt/tank/configs/immich1/data/library/`
1. `rsync -avhz --progress /mnt/tank/configs/immich/profile/ /mnt/tank/configs/immich1/data/profile/`
1. `rsync -avhz --progress /mnt/tank/configs/immich/thumbs/ /mnt/tank/configs/immich1/data/thumbs/`
1. `rsync -avhz --progress /mnt/tank/configs/immich/video/ /mnt/tank/configs/immich1/data/encoded-video/`
1. `rsync -avhz --progress /mnt/tank/configs/immich/uploads /mnt/tank/configs/immich1/data/upload`

Instead of running each individually, this will run sequentially but stop if any command fails to run. i.e. This runs the next command only if the previous one succeeds:
```bash
rsync -avhz --progress /mnt/tank/configs/immich/backups/ /mnt/tank/configs/immich1/data/backups/ && \
rsync -avhz --progress /mnt/tank/configs/immich/library/ /mnt/tank/configs/immich1/data/library/ && \
rsync -avhz --progress /mnt/tank/configs/immich/profile/ /mnt/tank/configs/immich1/data/profile/ && \
rsync -avhz --progress /mnt/tank/configs/immich/thumbs/ /mnt/tank/configs/immich1/data/thumbs/ && \
rsync -avhz --progress /mnt/tank/configs/immich/video/ /mnt/tank/configs/immich1/data/encoded-video/ && \
rsync -avhz --progress /mnt/tank/configs/immich/uploads/ /mnt/tank/configs/immich1/data/upload/

```
* Run rsync to sync the database:
```bash
rsync -avhz --progress /mnt/tank/configs/immich/db/ /mnt/tank/configs/immich1/db/
```

## Step 3 - Confirm data synced correctly
* `cd /mnt/tank/configs/immich1/data/thumbs` 
* `ls -l` 
* `cd /mnt/tank/configs/immich1/backups`
* `ls -l` 
* `cd /mnt/tank/configs/immich1/db`
* `ls -l`

## Step 4 - note the existing install config
* Grab the Database Password:
* Grab the Redis Password:

## Step 5 - Install Immich (again!)
* On TrueNAS:
* `Apps > Discover Apps`
* Search: "Immich"
* Click **Install Another Instance **
 
## Step 6 - Configure the New Instance
### Application name
* Application name: `immich1`
* Version: `1.9.103`
### Immich Configuration
* Enable Machine Learning: `checked`
* Machine Learning Image Type: `Default Machine Learning Image`
* Log Level: `Verbose`
* Database storage type: `HDD`
###  User and Group Configuration
* UserID: `568`
* GroupID: `568` 
### Network Configuration
* Port Number: `30042` (ensure this is unique - original install was 30041)
### Storage Configuration
* Use Old Storage Configuration (Deprecated): `Unchecked`
* Data Storage (aka Upload Location):
   * Type: `Host path`
   * Enable ACL: `unchecked`
   * Host Path: `/mnt/tank/configs/immich1/data`
* Machine Learning Cache: `Temporary`
* Postgres Data Storage:
   * Type: `Host path`
   * Enable ACL: `unchecked`
   * Host Path: `/mnt/tank/configs/immich1/db`
   * Automatic permissions: `checked`
### Resources Configuration
* CPUs: `4`
* Memory (in MB): `8000`

## Step 7 - Click install
Click **install**
