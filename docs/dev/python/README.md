# Python

```python
#!/usr/bin/python3                                                                                 │
                                                                                                   │       %S     second (00..60)
import time                                                                                        │
                                                                                                   │       %t     a tab
d = (int(time.time()))                                                                             │
                                                                                                   │       %T     time; same as %H:%M:%S
for i in range(0,30):                                                                              │
    print(d+i)
```

```python
#!/usr/bin/python3

for year in range(2001,2023):
    for month in range(1,12):
        for day in range(1,31):
            print(f'{year}'+f'{month}'+f'{day:02}')
```
