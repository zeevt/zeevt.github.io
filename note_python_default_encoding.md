This is just a note I want to be able to reference later.

```
C:\Users\zeev>c:\Python38\python.exe
>>> '😊'.encode()
b'\xf0\x9f\x98\x8a'
>>> with open('foo', 'w') as f:
...   f.write('😊')
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
  File "c:\Python38\lib\encodings\cp1252.py", line 19, in encode
    return codecs.charmap_encode(input,self.errors,encoding_table)[0]
UnicodeEncodeError: 'charmap' codec can't encode character '\U0001f60a' in position 0: character maps to <undefined>
>>>
>>> import locale
>>> locale.getpreferredencoding()
'cp1252'
>>> ^Z
C:\Users\zeev>set PYTHONUTF8=1

C:\Users\zeev>c:\Python38\python.exe
>>> import locale
>>> locale.getpreferredencoding()
'UTF-8'
>>> with open('foo', 'w') as f:
...   f.write('😊')
...
1
```

On Linux you get a similar issue (the portable C locale) if you don't have `LANG` environment variable set to something reasonable like `en_US.UTF-8`.
