# masto-pinb.py
Mastodon To Pinboard bookmark integration script

This is a Python script meant to be run repeatedly as a crontab job. It reads the latest toots from a [Mastodon](http://mastodon.social) account and bookmarks them in a [Pinboard.in](http://pinboard.in) account. 

```
usage: masto-pinb.py [-h] [--toots] [--log_json] [--favs] [--bmarks]
                     [--dry_run] [--verbose] [--get_last GET_LAST]

Arguments for Mastodon-Pinboard bookmarker app

optional arguments:
  -h, --help           show this help message and exit
  --toots              Bookmark user toots
  --log_json           Log toots in JSON format to local file
  --favs               Bookmark user favorites
  --bmarks             Bookmark user Mastodon bookmarks
  --dry_run            Dry run: don't actually bookmark anything
  --verbose            Print actions as they occur
  --get_last GET_LAST  retrieve only last n toots (default=20)
  ```

It is designed to be run periodically in a cron job. The period
between runs depends on your profligacy as a tooter and how much
latency you care about between when you post and when a toot is
bookmarked. Running this often will retrieve (but not bookmark)
redundant toots, but decrease latency; running it less often will
increase latency, but decrease API usage and possibly miss toots. You
may increase the GET_LAST value if you think you will generate more
than the default 20 toots returned by the API between runs. I am not a
heavy tooter so I use the `@hourly` crontab shortcut to run it hourly,
with the default 20 toot recall.

The Mastodon.social instance is rate-limited to something like 300
calls per 5 minutes. This script uses 3 calls at most per run; if the
rate limit is exceeded it will exit with an exception -- this is fine
as the script should pick up the unbookmarked posts on its next
invocation. The Pinboard rate limit is not documented that I could
find.
 
By default the script will bookmark all toots associated with the
account authorized in the `usercred.secret` file, as well as all toots
favorited and bookmarked by that user. This can be changed by any
combination of the `--toots`, `--bmarks`, and `--favs` command line
options; for example using only`--favs` will bookmark only favorites,
while `--toots --bmarks` wil bookmark toots and bookmarks but not
favorites.
 
Libraries and credentials for Pinboard and Mastodon
---

`pip3 install Mastodon.py` will install the required Mastodon Python libraries, 

`pip3 install "pinboard>=2.0"` will install thre required Pinboard libriaries.


`pip3 install html2txt` will install the required [html2text library](https://github.com/Alir3z4/html2text) by Aaron Swartz.

You will also need API tokens from both Pinboard and Mastodon. These are stored as text files with a `.secret` extension. (The `.secret` extension are included in the .gitignore so they are not checked into the repository). They should be kept in the same directory as the `masto-pinb.py` Python script.
  
  The Pinboard credentials can be obtained here [https://pinboard.in/settings/password](https://pinboard.in/settings/password) and should be stored as a single line of text in the file `pinboard_auth.secret`
  
The Mastodon user credential file is stored as `masto_pinb_usercred.secret` and can be generated by following the instructions at [https://mastodonpy.readthedocs.io/en/stable](https://mastodonpy.readthedocs.io/en/stable). You will need to register your app as part of the process but this need only be done once. 

Generated Files
---
Several files are generated in the same directory as the `masto-pinb.py` Python script.  In order to avoid duplicating Pinboard API calls, the IDs of recently-bookmarked toots are stored in the local text files `cached_bmarks.secret`, `cached_toots.secret`, and `cached_favs.secret`. (Though not particulalry sensitive, these have the `.secret` extension so that git will ignore them via `.gitignore`.) These files are truncated to the 120 most recent toots on every run so they will not grow large. 

If the `--log_json` command line argument is specified, every bookmarked toot is also stored locally, appended to the appropriate json file named `toots.json`, `favs.json`, and `bmarks.json`. The size of these filese is not managed and they may grow large. 
