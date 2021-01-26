# scripts

This repository contains a collection of bash scripts written to simplify everyday work.


## git-scrubber

*Tired of suspected bit flips corrupting your Git repositories? Look no further!*

git-scrubber is a bash script that looks in the supplied path(s) for Git
repositories and then executes the 'git fsck' command for each repository found.
The result can be logged to a supplied logfile. git-scrubber will help you
maintain the integrity of one, or typically many, Git repositories by
simplifying the task of verifying, i.e. scrubbing, repository integrity.

Typical use case for git-scrubber is to schedule periodic scrubbing of Git
repositories from the crontab and log results to file.

git-scrubber will place a file named "scrubber.lock" inside the respective Git
repository while checking it to prevent further scrubbing until current
scrubbing has finished. Scrubber will also try to execute using lowest possible
CPU and IO priority to minimize impact on system performance during scrubbing.


### Usage

```
git-scrubber [options] <pathspec>... [logfile]
```

### Options

```
-d, --debug
    Enable printing of debug information

-h, --help
    Display this help text.

-p, --print-heading
    Print a results heading to standard output and logfile. Will only print to
    file with the [-q|--quiet] option.

-q, --quiet
    Do not print results. Useful in automated use of scrubber where results are
    logged to file. Debug information will still be printed if the [-d|--debug]
    option is enabled.
```


### Example

Below is an example of git-scrubber usage with two git repositories 'linux' and 'foss' storing the result in a logfile named 'logfile':

```
git-scrubber -p /home/jeff/git/linux /home/jeff/git/foss /home/jeff/logfile
```

This would normally produce the following output to stdout and to the supplied logfile:

```
--------------------------------------------------------------------------------
TIME                 REPOSITORY                                DURATION  VERDICT
--------------------------------------------------------------------------------
2021-01-25T22:46:46  /home/jeff/git/linux/                        3 sec  PASSED
2021-01-25T22:46:49  /home/jeff/git/foss/                        10 sec  PASSED
```

If there are errors found in one of the repositories scrubbed the output could look like this:

```
--------------------------------------------------------------------------------
TIME                 REPOSITORY                                DURATION  VERDICT
--------------------------------------------------------------------------------
2021-01-25T22:46:46  /home/jeff/git/linux/                        3 sec  FAILED
2021-01-25T22:46:49  /home/jeff/git/foss/                        10 sec  PASSED
```

At this point it would be suitable to run a manual 'git fsck' iside the failed
repository and investigate the output. Restoring affected files from backup
would be a likely next step.

Subsequent runs of git-scrubber would skip scrubbing of the failed repo(s), showing 'LOCKED' status to avoid further touching the possibly corrupted files. To allow git-scrubber to scrub the respetive repos again simply remove the enclosing 'scrubber.lock' file(s).

```
--------------------------------------------------------------------------------
TIME                 REPOSITORY                                DURATION  VERDICT
--------------------------------------------------------------------------------
2021-01-25T23:46:46  /home/jeff/git/linux/                        3 sec  LOCKED
2021-01-25T23:46:49  /home/jeff/git/foss/                        10 sec  PASSED
```
