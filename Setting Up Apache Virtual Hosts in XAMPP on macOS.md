# Setting Up Apache Virtual Hosts in XAMPP on macOS

Here's how to create Apache Virtual Hosts in XAMPP on macOS:

## 1. Edit the hosts file

```bash
sudo nano /etc/hosts
```

Add your custom domains:
```
127.0.0.1       myproject.local
```

## 2. Configure httpd-vhosts.conf

Open the virtual hosts configuration file:
```bash
sudo nano /Applications/XAMPP/xamppfiles/etc/extra/httpd-vhosts.conf
```

Add your virtual host configuration:
```apache
<VirtualHost *:80>
    DocumentRoot "/Applications/XAMPP/xamppfiles/htdocs"
    ServerName localhost
</VirtualHost>

<VirtualHost *:80>
    ServerName myproject.local
    DocumentRoot "/Applications/XAMPP/xamppfiles/htdocs/myproject"
    <Directory "/Applications/XAMPP/xamppfiles/htdocs/myproject">
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog "logs/myproject.local-error_log"
    CustomLog "logs/myproject.local-access_log" common
</VirtualHost>
```

## 3. Verify httpd.conf includes vhosts file

Make sure the main configuration file includes the vhosts file:
```bash
sudo nano /Applications/XAMPP/xamppfiles/etc/httpd.conf
```

Check that this line is uncommented (no # at the start):
```apache
Include etc/extra/httpd-vhosts.conf
```

## 4. Restart Apache

Restart Apache using the XAMPP Control Panel or with this command:
```bash
sudo /Applications/XAMPP/xamppfiles/xampp restart
```

Now you should be able to access your site at `http://myproject.local` in your browser.