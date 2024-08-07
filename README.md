# Fork

This is a fork of [Duplicacy](https://github.com/gilbertchen/duplicacy/) for my own personal use.
If you want to use Duplicacy, I recommend getting the official release.

# Purpose

This fork has one purpose. To introduce a new function that I needed for data-recovery purposes. 
Nothing else was modified except for adding this new function and modifying visibility of certain
properties in order to fully realize the necessary functionality.

# Re-Upload (and encrypt) Snapshots to Storage

## Usage

Ideally make a backup of the "snapshots" folder in both your local duplicacy cache and your remote
storage folder just in case this messes something up.

```
duplicacy reupload-snapshot-file -storage STORAGE
```

## Background

I inadvertantly deleted all of my snapshot files from my S3/Wasabi storage location.

The only backups I had were in my local duplicacy folder (`~/.duplicacy/cache/<STORAGE>/snapshots/<SNAPSHOT ID>`).

The problem is the local copies are unencrypted (raw JSON) and the files in the storage `/snapshots/` folder are encrypted.

As near as I can tell, the closest built in option for solving my problem would be the `copy` command, but it doesn't
appear to support two storages with different settings (e.g. encryption), so I wrote this to accomplish my needs.

The process simply:

- Scan all local snapshot files in the specified storage's default snapshot ID (from the cache folder)
- Scan all remote snapshot files in the specified storage
- For each file that is present in cache but not present in the storage
    - encrypt and upload that file
    
## Improvement

It would be helpful to have it check the remote storage files and if they are unencrypted, encrypt them.

This shouldn't happen in normal use-case, but in my case I didn't realize the local cache files were unencrypted
and attempted to recover by copying those cache files to my storage. This just lead to backups failing due to the
revision files being "corrupted".

# Other enhancement ideas

## Rename snapshot IDs

Snapshot files are encrypted with their path as part of the encryption key. This means you can't easily rename
a snapshot ID. 

Using the techniques from this code it would be easy to download the snapshot files from the remote storage, decrypt,
rename to the new snapshot ID, re-encrypt, and re-upload. Everything else would work just fine, as the chunks referenced
in the snapshot file would continue to be the same and wouldn't need any re-encryption.