# WordPress Stack with Flox

A complete WordPress development environment using [Flox](https://flox.dev) with MySQL, PHP-FPM, and Nginx.

## Stack Components

- **WordPress** - Latest version downloaded and configured
- **MySQL 8.0** - Database server with custom data directory
- **PHP 8.2** - With PHP-FPM for processing
- **Nginx** - Web server with WordPress-optimized configuration

## Quick Start

```bash
# Clone/navigate to project directory
cd flox-wpstack

# Activate the environment (auto-downloads and sets up everything)
flox activate

# Start all services
flox services start

# Access WordPress
open http://localhost:8080
```

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Nginx       │    │    PHP-FPM      │    │     MySQL       │
│   (Port 8080)   │───▶│   (Port 9000)   │───▶│   (Port 3306)   │
│                 │    │                 │    │                 │
│ Static files &  │    │ WordPress PHP   │    │ Database:       │
│ Reverse proxy   │    │ execution       │    │ wordpress       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Database Configuration

- **Host**: `127.0.0.1` (TCP connection)
- **Database**: `wordpress`
- **User**: `root`
- **Password**: `wordpress`
- **Port**: `3306`

## Troubleshooting

### Database Connection Issues

If you encounter MySQL connection errors, here are the most common issues and solutions:

#### 1. "No such file or directory" Error

**Error**: `mysqli_real_connect(): (HY000/2002): No such file or directory`

**Cause**: WordPress is trying to use Unix socket connection with `localhost`, but the socket is in a custom location.

**Solution**: Use TCP connection instead
- Ensure `WORDPRESS_DB_HOST = "127.0.0.1"` in `.flox/env/manifest.toml`
- Restart environment: `flox activate`

#### 2. "Access denied for user 'root'@'localhost'" Error

**Error**: `Access denied for user 'root'@'localhost' (using password: YES)`

**Cause**: MySQL treats `localhost` and `127.0.0.1` as different hosts. Permissions were only granted for `localhost`.

**Solution**: Reset database or fix permissions

**Option A - Reset Database (Recommended):**
```bash
# Stop services
flox services stop

# Remove MySQL data directory to force clean re-initialization
rm -rf .flox/cache/mysql/data

# Restart with clean database
flox activate
flox services start
```

**Option B - Fix Permissions Manually:**
```bash
mysql -u root -e "
CREATE USER IF NOT EXISTS 'root'@'127.0.0.1' IDENTIFIED BY 'wordpress';
GRANT ALL PRIVILEGES ON wordpress.* TO 'root'@'127.0.0.1';
FLUSH PRIVILEGES;
"
```

#### 3. Services Not Starting

**Check service status:**
```bash
flox services status
```

**Restart services:**
```bash
flox services stop
flox services start
```

**Check logs:**
```bash
# Check individual service logs in
ls .flox/cache/*/
```

## File Structure

```
flox-wpstack/
├── .flox/
│   ├── env/
│   │   └── manifest.toml          # Flox environment configuration
│   └── cache/
│       ├── mysql/                 # MySQL data directory and config
│       ├── nginx/                 # Nginx config and logs
│       └── php-fpm/              # PHP-FPM config
├── wordpress/                     # WordPress installation
│   ├── wp-config.php             # Auto-generated WordPress config
│   └── [WordPress files...]
├── latest.tar.gz                 # WordPress download archive
└── README.md
```

## Development Workflow

1. **Start development:**
   ```bash
   flox activate
   flox services start
   ```

2. **Access your site:**
   - WordPress: http://localhost:8080
   - Direct PHP files: Place in `wordpress/` directory

3. **Database access:**
   ```bash
   mysql -u root -pwordpress wordpress
   ```

4. **Stop when done:**
   ```bash
   flox services stop
   ```

## Configuration Details

### WordPress Configuration
- Debug mode enabled (`WP_DEBUG = true`)
- Database connection via TCP (`127.0.0.1`)
- UTF-8 charset

### Nginx Configuration
- Listens on port 8080
- Serves static files directly
- Proxies PHP requests to PHP-FPM
- WordPress-friendly URL rewriting

### PHP-FPM Configuration
- Listens on `127.0.0.1:9000`
- Dynamic process management
- Optimized for development

### MySQL Configuration
- Custom data directory in `.flox/cache/mysql/data`
- Socket file in `.flox/cache/mysql/mysql.sock`
- Permissions for both `localhost` and `127.0.0.1`

## Environment Variables

Key variables defined in `.flox/env/manifest.toml`:

```toml
[vars]
MYSQL_HOST = "127.0.0.1"
MYSQL_PORT = "3306"
MYSQL_USER = "root"
MYSQL_PASSWORD = "wordpress"
MYSQL_DATABASE = "wordpress"

WORDPRESS_DB_HOST = "127.0.0.1"
WORDPRESS_DB_NAME = "wordpress"
WORDPRESS_DB_USER = "root"
WORDPRESS_DB_PASSWORD = "wordpress"
```

## Tips

- **Clean restart**: Remove `.flox/cache/` to reset all service data
- **WordPress updates**: Delete `wordpress/` and `latest.tar.gz`, then `flox activate`
- **Database reset**: Delete `.flox/cache/mysql/data/` and restart services
- **Logs**: Check `.flox/cache/*/` directories for service-specific logs

## Requirements

- [Flox](https://flox.dev) installed on your system
- Internet connection for initial package downloads

## License

This is a development environment template. WordPress itself is licensed under GPL v2 or later.