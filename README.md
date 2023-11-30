# Bug Report

This repository provides a reproducible example of iterative/dvc#10124

## Description

push fails when a tracked directory is missing from the local cache.  

I have a use case where an automated process runs `dvc repro` for a subset of the DAG stages in a clean checkout of the git repository.  The `dvc.lock` file references outputs from prior pipeline runs in the project that are not present in the workspace or cache, but I do not want to regenerate or pull into the workspace because they are stored in a separate remote that can't be accessed from the environment.  Prior to version 3.27.1, I could run my pipeline and `dvc push` and only receive a warning about the missing files.  Now the process fails and no data is pushed to the remote.

From developing a reproducible example, it appears to me this only impacts tracked directories, and not regular files.

### Reproduce

Requires [poetry](https://python-poetry.org/)

```
$ git clone git@github.com:ahasha/dvc-push-issue.git
$ mkdir /tmp/dvc_push_issue_remote/
$ dvc --version
3.27.1
$ dvc push
ERROR: unexpected error - failed to load directory ('00', '94fc824a413b994ecf30756dcb6f17.dir'): [Errno 2] No such file or directory: '/Users/alexhasha/repos/dvc-push-issue/.dvc/cache/files/md5/00/94fc824a413b994ecf30756dcb6f17.dir'
$ echo $?
255
```

Downgrading to 3.27.0 changes the behavior.
```
$ git checkout 414053a5e7d6e1c96
$ poetry install
Installing dependencies from lock file

Package operations: 0 installs, 2 updates, 0 removals

  • Downgrading dvc-data (2.19.0 -> 2.18.2)
  • Downgrading dvc (3.27.1 -> 3.27.0)
$ dvc --version
3.27.0
$ dvc push
WARNING: Some of the cache files do not exist neither locally nor on remote. Missing cache files:                                   
name: None, md5: 0094fc824a413b994ecf30756dcb6f17.dir
WARNING: Some of the cache files do not exist neither locally nor on remote. Missing cache files:                                   
name: data/foo.txt, md5: 8fdd769621e003fe3c0c21e9929b491e
Everything is up to date.
$ echo $?
0
```

I have also tested and confirmed the error still occurs in versions 3.30.3 and 3.30.1, with corresponding commits in the repo.

### Expected

In version 3.27.0 and prior, this situation resulted in a warning, but the push completed successfully and returned exit code 0.  I would like to see this behavior restored.

### Environment information

```
DVC version: 3.27.1 (pip)
-------------------------
Platform: Python 3.11.6 on macOS-14.1.1-x86_64-i386-64bit
Subprojects:
	dvc_data = 2.19.0
	dvc_objects = 1.3.0
	dvc_render = 0.6.0
	dvc_task = 0.3.0
	scmrepo = 1.5.0
Supports:
	http (aiohttp = 3.9.1, aiohttp-retry = 2.8.3),
	https (aiohttp = 3.9.1, aiohttp-retry = 2.8.3)
Config:
	Global: /Users/alexhasha/Library/Application Support/dvc
	System: /Library/Application Support/dvc
Cache types: <https://error.dvc.org/no-dvc-cache>
Caches: local
Remotes: local
Workspace directory: apfs on /dev/disk1s3s1
Repo: dvc, git
Repo.site_cache_dir: /Library/Caches/dvc/repo/86773253b3d9be97d0d2bb3e38c01565
```
