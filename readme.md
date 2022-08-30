# ColdFusion / Lucee lnav log format

[lnav](https://lnav.org/) is a wonderful log file viewer in terminal for Mac and Linux. This repo provides a CFML log format definition file for lnav so that you can use lnav properly to view CF & Lucee log files.

To use, download the `cflog.json` definition file from the `lnav-format` folder then install it with lnav with the following command:
> `lnav -i cflog.json`

# lnav

[lnav documentation](https://docs.lnav.org/en/latest/intro.html)

lnav is powerful. To take advantage of its full features, refer to the documentations. Below is a partial list of what it can do:

- `lnav /path/to/log/folder` to view all log files in a folder, or `lnav /path/to/log/file.log` to view a single log file
- Basic log content navigation: `Left`/`Right`, `PgUp`/`PgDn`, `Home`/`End`
- Note: for some hotkeys, use it with `shift` to do the opposite function (e.g. `f` to jump to the next file and `shift + f` to jump to previous file)
- `ctrl + w` to toggle word-wrap
- `p` to show log line parsing result (display the log line into items so they are easier to read)
- `shift` + `p` to format the log content for better display or printing (for example if you have json or xml in log, it will format them with proper indention)
- Navigate through log base on time period: `1`=10 mins, `2`=20 mins, .., `6`=60 mins, `d`=24hrs
- `i` to show histogram
- `m` to bookmark the line, and then use `u` to go through bookmarks
- `/` to start searching with regexp, then use `n` to go through search result line
- `:` to trigger command mode
  - `:filter-in` and `:filter-out` to filter the log content
  - `:comment` to comment on a line, `clear-comment` to remove
  - `:tag` to tag a line, `un-tag` to remove, `delete-tags` to remove tag(s)
  - `:save-session` to save the current stage/config to a named session, and use `:load-session` to load and apply it
- lnav will use a session to remember your commands such filter-in/out and markers. You can use `ctrl + r` to reset the session
- `;` to run SQL query on the log data. Do a `select * from cflog` to see what column names we can use for query, some of them are:
  - `log_time`, `log_level`, `log_mark`, `log_comment`, `log_tag`
- Use `tab` to go into filter panel (use `q` to exist the panel), and while in the filter panel:
  - `i` to add a filter-in, `o` to add a filter-out, `t` to flip your filter (filter IN <-> filter OUT), `spacebar` to enable/disable the filter
- If the views contains multiple files, use `f` to go to next file
- Change theme (default themes are: default, eldar, grayscale, monocai, night-owl, solarized-dark, and solarized-light)
  - `:config /ui/theme/ <theme_name>`

# Writing a lnav CF log format file

If you want to write you own log format file, see [lnav log formats docs](https://docs.lnav.org/en/latest/formats.html). Below is a brief explanation on how the one in this repo is written.

### Crafting the regexp pattern

The regexp pattern in the format definition file might look massive, but it is mostly due to we need to escape all the double quotes from CF log files and the regexp character classes (such as `\d`) within json. Below is an explanation on how the one in this repo is constructed.

A CF log line can be something likes:
> `"INFO","XNIO-1 task-2","05/18/2022","09:48:55","lucee.runtime.CFMLFactoryImpl","Reset 6 Unused PageContexts"`

So the regexp to parse a CF log line would be likes:
> `^"([A-Z]+)","([^\"]+)","(\d{2}\/\d{2}\/\d{4}","\d{2}:\d{2}:\d{2})","([^\"]+)","([^\"]+)"$`

The regexp will give us 5 groups, from the line example above they are:
1. INFO
2. XNIO-1 task-2
3. 05/18/2022","09:48:55
4. lucee.runtime.CFMLFactoryImpl
5. Reset 6 Unused PageContexts

Now we can add our lnav fields into the regexp pattern. The lnav fields are written as `?<field>` and will be placed before the group regexp. The fields we used for the 5 groups matched by regexp are:
1. `?<severity>` - Our custom field name for `level`, if you use a custom name then you must define it in the `level-field` item (see the section "Items" below)
2. `?<thread_id>` - Our custom field name
3. `?<timestamp>` - This is the field lnav used to parse date & time, you must use this name exactly
4. `?<application>` - Our custom field name
5. `?<body>` - Our custom field name (most formats use `body`, so we keep it this way)

So the final regexp pattern for lnav will be:
> `^"(?<severity>[A-Z]+)","(?<thread_id>[^"]+)","(?<timestamp>\d{2}\/\d{2}\/\d{4}","\d{2}:\d{2}:\d{2})","(?<application>[^"]+)","(?<body>[^"]+)"$`

And to use that pattern string in a json file you need to escape double quotes and slashes, hence the final pattern string will be:
> `^\"(?<severity>[A-Z]+)\",\"(?<thread_id>[^\"]+)\",\"(?<timestamp>\\d{2}\/\\d{2}\/\\d{4}\",\"\\d{2}:\\d{2}:\\d{2})\",\"(?<application>[^\"]+)\",\"(?<body>[^\"]+)\"$`

Note: for the `?<timestamp>` field, the matched value contains date & time value in the format likes `dd/mm/yyyy","hh:mm:ss` (notice the double quotes and comma between the date & time value), so in the `timestamp-format` item (see the section "Items" below) we can tell lnav how to properly parse the date and time out of this string as `%m/%d/%Y\",\"%H:%M:%S`

### Other items

- `level-field`
  - This defines the column name from your regexp pattern that lnav will use to determine the alert level value (i.e. Info, Warning, Error, Fatal etc).
- `level`
  - A structure defines the alert levels and their corresponding text value in the column
- `value`
  - A structure defines the column types you used in the regexp pattern.
- `timestamp-format`
  - A string tells lnav how to parse the date & time from the `?<timestamp>` column value it get from the regexp pattern.
- `sample`
  - A log line example. If lnav have trouble parsing this sample then it won't load the log files and will throw an error telling you what went wrong.

### See what other format files are written

- https://github.com/hagfelsh/lnav_formats.git
- https://github.com/PaulWay/lnav-formats.git
- https://github.com/penntaylor/lnav-ruby-logger-format.git
- https://github.com/aspiers/lnav-formats.gi

# ToDo

- Improve the log format definition regular express
  - It currently cannot handel the log body message that contains double quote