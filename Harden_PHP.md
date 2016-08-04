#Harden your PHP configuration

#PHP.ini
This is a compilation of some best configuration for php.ini security
Some may change depending on the project we are working on at `innoveos`

```php

# Disallow dangerous functions
disable_functions = php_uname, getmyuid, getmypid, passthru,
leak, listen, diskfreespace, tmpfile, link, ignore_user_abord, shell_exec, dl, set_time_limit, 
exec, system, highlight_file, source, show_source, fpaththru, virtual, posix_ctermid, posix_getcwd,
posix_getegid, posix_geteuid, posix_getgid, posix_getgrgid, posix_getgrnam, posix_getgroups, posix_getlogin,
posix_getpgid, posix_getpgrp, posix_getpid, posix, _getppid, posix_getpwnam, posix_getpwuid, posix_getrlimit,
posix_getsid, posix_getuid, posix_isatty, posix_kill, posix_mkfifo, posix_setegid, posix_seteuid, posix_setgid,
posix_setpgid, posix_setsid, posix_setuid, posix_times, posix_ttyname, posix_uname, proc_open, proc_close,
proc_get_status, proc_nice, proc_terminate, phpinfo 

## Try to limit resources  ##
 
# Maximum execution time of each script, in seconds
max_execution_time = 30
 
# Maximum amount of time each script may spend parsing request data
max_input_time = 60
 
# Maximum amount of memory a script may consume (8MB)
memory_limit = 8M
 
# Maximum size of POST data that PHP will accept.
post_max_size = 8M
 
# Whether to allow HTTP file uploads.
# This is depends on the project.
# file_uploads = Off
 
# Maximum allowed size for uploaded files.
upload_max_filesize = 2M
 
# Do not expose PHP error messages to external users
display_errors = Off
 
# Turn on safe mode
safe_mode = On
 
# Only allow access to executables in isolated directory
safe_mode_exec_dir = php-required-executables-path
 
# Limit external access to PHP environment
safe_mode_allowed_env_vars = PHP_
 
# Restrict PHP information leakage
expose_php = Off
 
# Log all errors
log_errors = On
 
# If On, all variables passed through GET or POST are available as global variables our script
# so disable it 
register_globals = Off
 
# Minimize allowable PHP post size
post_max_size = 1K
 
# Ensure PHP redirects appropriately
cgi.force_redirect = 0
 
# Disallow uploading unless necessary
file_uploads = Off
 
# Enable SQL safe mode(Disabled bcz causes some errors with Word Press)
#sql.safe_mode = On
 
# Avoid Opening remote files by including malicious PHP scripts
allow_url_fopen = Off
allow_url_include= Off

# Escape some bad characters
# magic_quotes_gpc = On

# Disable X-Php Originating
mail.add_x_header= Off

# Session's security  : More at official (http://php.net/manual/en/session.configuration.php )

# To avoid some XSS attacks
session.cookie_httponly = 1

# Cookies will only be sent over an HTTPS only
session.cookie_secure = 1

# Change default location of session 
session.save_path = /var/lib/php

#Change session ID name to spoof ASP.net language
session.name = ASP.NET_SessionId

```

