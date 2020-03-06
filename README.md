# jtbl
A simple cli tool to print JSON data as a table in the terminal.

`jtbl` accepts piped JSON data from `stdin` and outputs a text table representation to `stdout`. e.g:
```
$ cat cities.json | jtbl 
  LatD    LatM    LatS  NS      LonD    LonM    LonS  EW    City               State
------  ------  ------  ----  ------  ------  ------  ----  -----------------  -------
    41       5      59  N         80      39       0  W     Youngstown         OH
    42      52      48  N         97      23      23  W     Yankton            SD
    46      35      59  N        120      30      36  W     Yakima             WA
    42      16      12  N         71      48       0  W     Worcester          MA
    43      37      48  N         89      46      11  W     Wisconsin Dells    WI
    36       5      59  N         80      15       0  W     Winston-Salem      NC
    49      52      48  N         97       9       0  W     Winnipeg           MB
```

`jtbl` expects a JSON array of JSON objects or JSON lines.

It can be useful to JSONify command line output with `jc`, filter through `jq`, and present in `jtbl`:
```
$ jc ifconfig | jq -c '.[] | {name, type, ipv4_addr, ipv4_mask}'| jtbl 
name     type            ipv4_addr       ipv4_mask
-------  --------------  --------------  -------------
docker0  Ethernet        172.17.0.1      255.255.0.0
ens33    Ethernet        192.168.71.146  255.255.255.0
lo       Local Loopback  127.0.0.1       255.0.0.0
```
> Notice the `-c` flag used in `jq` to produce valid JSON lines output for `jtbl` consumption.

## Installation
```
pip3 install --upgrade jtbl
```
## Usage
Just pipe JSON data to `jtbl`. (e.g. `cat` a JSON file, `jc`, `jq`, `aws` cli, `kubectl`, etc.)
```
$ <JSON Source> | jtbl
```
## Compatible JSON Formats
`jtbl` works best with a shallow array of JSON objects. Each object should have a few elements that will be turned into table columns. Fortunately, this is how many APIs present their data.

**JSON Array Example**
```
[
  {
    "unit": "proc-sys-fs-binfmt_misc.automount",
    "load": "loaded",
    "active": "active",
    "sub": "waiting",
    "description": "Arbitrary Executable File Formats File System Automount Point"
  },
  {
    "unit": "sys-devices-pci0000:00-0000:00:07.1-ata2-host2-target2:0:0-2:0:0:0-block-sr0.device",
    "load": "loaded",
    "active": "active",
    "sub": "plugged",
    "description": "VMware_Virtual_IDE_CDROM_Drive"
  },
  ...
]
```

`jtbl` can also work with [JSON Lines](http://jsonlines.org/) format with similar features.

**JSON Lines Example**
```
{"name": "docker0", type": "Ethernet", "ipv4_addr": "172.17.0.1", "ipv4_mask": "255.255.0.0"}
{"name": "ens33", "type": "Ethernet", "ipv4_addr": "192.168.71.146", "ipv4_mask": "255.255.255.0"}
{"name": "lo", "type": "Local Loopback", "ipv4_addr": "127.0.0.1", "ipv4_mask": "255.0.0.0"}
```

## Filtering the JSON Input
If there are too many elements, or the data in the elements are too large, the table may not fit in the terminal screen. In this case you can use a JSON filter like `jq` to send `jtbl` only the elements you are interested in:

The following example uses `jq` to filter and then again to 'slurp' the filtered elements into a proper JSON array.

**`jq` Slurp Method**
```
$ cat /etc/passwd | jc --passwd | jq '.[] | {username, shell}' | jq -s
[
  {
    "username": "root",
    "shell": "/bin/bash"
  },
  {
    "username": "bin",
    "shell": "/sbin/nologin"
  },
  {
    "username": "daemon",
    "shell": "/sbin/nologin"
  },
  ...
]
```

Or you can use the `-c` option in `jq` to send the data compact, which will effectively be JSON Lines format, which `jtbl` can understand:

**`jq` JSON Lines Method**
```
$ cat /etc/passwd | jc --passwd | jq -c '.[] | {username, shell}'
{"username":"root","shell":"/bin/bash"}
{"username":"bin","shell":"/sbin/nologin"}
{"username":"daemon","shell":"/sbin/nologin"}
...
```

When piping either of these to `jtbl` you get the following result:
```
username         shell
---------------  --------------
root             /bin/bash
bin              /sbin/nologin
daemon           /sbin/nologin
...
```

## Working with Deeper JSON Structures
`jtbl` will happily dump deeply nested JSON structures into a table, but usually this is not what you are looking for.
```
$ jc dig www.cnn.com | jtbl 
   id  opcode    status    flags                query_num    answer_num    authority_num    additional_num  question           answer               query_time    server  when                 rcvd
-----  --------  --------  -----------------  -----------  ------------  ---------------  ----------------  -----------------  -----------------  ------------  --------  -----------------  ------
25618  QUERY     NOERROR   ['qr', 'rd', 'ra'            1             5                0                 1  {'name': 'www.cnn  [{'name': 'www.cn            34      2600  Fri Mar 06 07:25:     143
                           ]                                                                                .com.', 'class':   n.com.', 'class':                          23 PST 2020
                                                                                                            'IN', 'type': 'A'   'IN', 'type': 'C
                                                                                                            }                  NAME', 'ttl': 264
                                                                                                                               , 'data': 'turner
                                                                                                                               -tls.map.fastly.n
                                                                                                                               et.'}, {'name': '
                                                                                                                               turner-tls.map.fa
                                                                                                                               stly.net.', 'clas
                                                                                                                               s': 'IN', 'type':
                                                                                                                                'A', 'ttl': 13,
                                                                                                                               'data': '151.101.
                                                                                                                               129.67'}, {'name'
                                                                                                                               : 'turner-tls.map
                                                                                                                               .fastly.net.', 'c
                                                                                                                               lass': 'IN', 'typ
                                                                                                                               e': 'A', 'ttl': 1
                                                                                                                               3, 'data': '151.1
                                                                                                                               01.65.67'}, {'nam
                                                                                                                               e': 'turner-tls.m
                                                                                                                               ap.fastly.net.',
                                                                                                                               'class': 'IN', 't
                                                                                                                               ype': 'A', 'ttl':
                                                                                                                                13, 'data': '151
                                                                                                                               .101.1.67'}, {'na
                                                                                                                               me': 'turner-tls.
                                                                                                                               map.fastly.net.',
                                                                                                                                'class': 'IN', '
                                                                                                                               type': 'A', 'ttl'
                                                                                                                               : 13, 'data': '15
                                                                                                                               1.101.193.67'}]
```

To get to the data you are interested in you can use a JSON filter like `jq` do dive deeper.

**Diving Deeper into the JSON with `jq`**
```
$ jc dig www.cnn.com | jq '.[].answer' 
[
  {
    "name": "www.cnn.com.",
    "class": "IN",
    "type": "CNAME",
    "ttl": 90,
    "data": "turner-tls.map.fastly.net."
  },
  {
    "name": "turner-tls.map.fastly.net.",
    "class": "IN",
    "type": "A",
    "ttl": 20,
    "data": "151.101.197.67"
  }
]
```
This will produce the following table in `jtbl`
```
name               class    type      ttl  data
-----------------  -------  ------  -----  -----------------
www.cnn.com.       IN       CNAME      72  turner-tls.map.fa
                                           stly.net.
turner-tls.map.fa  IN       A          17  151.101.1.67
stly.net.
turner-tls.map.fa  IN       A          17  151.101.193.67
stly.net.
turner-tls.map.fa  IN       A          17  151.101.65.67
stly.net.
turner-tls.map.fa  IN       A          17  151.101.129.67
stly.net.
```
## Column Width
Today `jtbl` will always wrap each column at 17 characters. In the future, `jtbl` will be smarter and only wrap when necessary and  also provide the option to truncate vs. wrap lines. Stay tuned!