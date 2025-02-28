---
title: How to use the 'blobfuse2 unmount' command to unmount an existing mount point (preview)| Microsoft Docs
titleSuffix: Azure Storage
description: How to use the 'blobfuse2 unmount' command to unmount an existing mount point. (preview)
author: jimmart-dev
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.date: 10/17/2022
ms.author: jammart
ms.reviewer: tamram
---

# How to use the BlobFuse2 unmount command (preview)

Use the `blobfuse2 unmount` command to unmount one or more existing BlobFuse2 mount points.

[!INCLUDE [storage-blobfuse2-preview](../../../includes/storage-blobfuse2-preview.md)]

## Syntax

The `blobfuse2 unmount` command has 2 formats:

`blobfuse2 unmount [mount path] [flags]`

`blobfuse2 unmount all [flags]`

## Arguments

`[mount path]`

Specify a file path to the directory that contains the mount point to be unmounted. Example:

```bash
blobfuse2 unmount ./mount_path ...
```

`all`

Unmount all existing BlobFuse2 mount points.

## Flags (options)

The following flags were inherited from the parent (the BlobFuse2 command):

| Flag | Short version | Value type | Default value | Example | Description |
|--|--|--|--|--|--|
| disable-version-check |    | boolean | false | --disable-version-check=true | Enables or disables automatic version checking of the BlobFuse2 binaries |
| help                  | -h | n/a     |       | -h or --help                 | Help info for the blobfuse2 command and subcommands                      |

There are no flags only supported by the unmount command.

## Examples

1. Unmount a BlobFuse2 mount instance:

    ```bash
    blobfuse2 unmount ./mount_path
    ```

    (Alternatively, you can use a native Linux command to do the same):

    ```bash
    sudo fusermount3 -u ./mount_path
    ```

1. Unmount all BlobFuse2 mount points (see also [The BlobFuse2 unmount all command](blobfuse2-commands-unmount-all.md)):

    ```bash
    blobfuse2 unmount all
    ```

## See also

- [The Blobfuse2 unmount all command (preview)](blobfuse2-commands-unmount-all.md)
- [The Blobfuse2 mount command (preview)](blobfuse2-commands-mount.md)
