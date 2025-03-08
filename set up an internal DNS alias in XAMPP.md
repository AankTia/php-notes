To set up an internal DNS alias in XAMPP so you can use custom domain names for your local development, you need to modify a couple of files:

## Method 1: Using the hosts file

1. Open your system's hosts file:
   - **Windows**: Open Notepad as Administrator and open `C:\Windows\System32\drivers\etc\hosts`
   - **macOS/Linux**: Open Terminal and run `sudo nano /etc/hosts`

2. Add your custom domain(s) by adding lines like:
   ```
   127.0.0.1    myproject.local
   127.0.0.1    another-project.test
   ```

3. Save the file.

## Method 2: Using Apache Virtual Hosts

1. Open the Apache configuration file:
   - In XAMPP, edit: `xampp/apache/conf/extra/httpd-vhosts.conf`

2. Add your virtual host configurations:
   ```apache
   <VirtualHost *:80>
       ServerName myproject.local
       DocumentRoot "C:/xampp/htdocs/myproject"
       <Directory "C:/xampp/htdocs/myproject">
           Options Indexes FollowSymLinks MultiViews
           AllowOverride All
           Require all granted
       </Directory>
   </VirtualHost>
   ```

3. Save the file and restart Apache from the XAMPP control panel.

Make sure to combine both methods - add the domain to your hosts file AND configure the virtual host in Apache.

After making these changes, you can access your project at `http://myproject.local` in your browser instead of `http://localhost/myproject`.