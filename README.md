# 🚀 Docker Compose Environment — LiteSpeed + MariaDB + Laravel

![Docker](https://img.shields.io/badge/Docker-✓-2496ED?style=flat-square&logo=docker)
![LiteSpeed](https://img.shields.io/badge/LiteSpeed-✓-007ACC?style=flat-square)
![Laravel](https://img.shields.io/badge/Laravel-✓-FF2D20?style=flat-square&logo=laravel)
![MariaDB](https://img.shields.io/badge/MariaDB-✓-003545?style=flat-square&logo=mariadb)

A complete, production-like Docker-based development stack for Laravel applications using **LiteSpeed Web Server**, **MariaDB**, and a custom **Ubuntu frontend container**.

## ✨ Features

- **High-Performance Stack**: LiteSpeed Web Server with LS PHP 8.1
- **Laravel-Optimized**: Pre-configured for Laravel applications
- **SSL Ready**: Built-in ACME support for Let's Encrypt certificates
- **Admin Panel**: LiteSpeed WebAdmin interface included
- **Persistent Data**: All data survives container restarts
- **Auto Database Setup**: SQL scripts automatically executed on first run
- **Development-Friendly**: Hot-reload ready for Laravel development

## 📦 Service Architecture

### 1. 🖥️ Frontend Container (`frontend`)
**Base Image**: Ubuntu 22.04  
**Purpose**: Application file management and initialization

| Feature | Description |
|---------|-------------|
| **Laravel Files** | Persistent volume for Laravel application |
| **SQL Scripts** | Initial database setup scripts |
| **Runtime** | Kept alive with `tail -f /dev/null` |
| **Linked** | Directly connected to LiteSpeed container |

**Volumes:**
```yaml
./frontend/laravel:/home/ubuntu/frontend/laravel
./frontend/basedatos-inicio:/home/ubuntu/frontend/basedatos-inicio
```

---

### 2. ⚡ LiteSpeed Web Server (`lsws`)
**Custom Build**: From `./docker-customs/lsws/template`  
**PHP**: LS PHP 8.1

**Features:**
- Serves Laravel from `/var/www/vhosts/laravel`
- Custom configuration support
- Built-in admin panel
- ACME SSL certificate automation
- Virtual hosts management

**Port Mapping:**
| Port | Purpose | URL |
|------|---------|-----|
| `80:80` | HTTP | `http://localhost` |
| `443:443` | HTTPS | `https://localhost` |
| `7080:7080` | Admin Panel | `http://localhost:7080` |

**Volumes:**
```yaml
./frontend/laravel:/var/www/vhosts/laravel
./lsws/conf:/usr/local/lsws/conf
./lsws/admin/conf:/usr/local/lsws/admin/conf
./bin/container:/usr/local/bin
./sites:/var/www/vhosts/
./acme:/root/.acme.sh/
./logs:/usr/local/lsws/logs/
```

---

### 3. 🗄️ MariaDB Database (`mariadb`)
**Purpose**: Laravel application database

**Environment Variables:**
```env
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=laravel
MYSQL_USER=laravel
MYSQL_PASSWORD=laravel
```

**Port:** `3306:3306`

**Volumes:**
```yaml
./mariadb:/var/lib/mysql
./frontend/basedatos-inicio/init.sql:/docker-entrypoint-initdb.d/init.sql
```

**Note:** The `init.sql` file is automatically executed on first container run.

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose installed
- Git (optional)

### Installation

1. **Clone and navigate:**
   ```bash
   git clone <your-repo-url>
   cd <project-directory>
   ```

2. **Start the stack:**
   ```bash
   docker compose up -d
   ```

3. **Access services:**
   - **LiteSpeed Admin**: http://localhost:7080
   - **Application**: http://localhost
   - **Database**: `localhost:3306`

4. **Deploy Laravel:**
   Place your Laravel application in:
   ```
   frontend/laravel/
   ```

## 📁 Project Structure

```bash
project/
├── 📄 docker-compose.yaml           # Main compose file
├── 📁 frontend/
│   ├── 📁 laravel/                  # 🎯 Laravel application files
│   └── 📁 basedatos-inicio/         # 📊 Initial SQL scripts
├── 📁 mariadb/                      # 🗄️ Database persistent data
├── 📁 lsws/
│   ├── 📁 conf/                     # ⚙️ LiteSpeed configuration
│   └── 📁 admin/conf/               # 👨‍💼 Admin panel config
├── 📁 docker-customs/
│   └── 📁 lsws/template             # 🏗️ Custom Dockerfile for LSWS
├── 📁 sites/                        # 🌐 Virtual hosts directory
├── 📁 logs/                         # 📝 Web server logs
└── 📁 acme/                         # 🔒 SSL certificates (ACME)
```

## 🔧 Configuration

### Customizing LiteSpeed
1. Access the admin panel at `http://localhost:7080`
2. Default credentials: `admin` / `[check generated password]`
3. Modify virtual hosts, PHP settings, and SSL configurations

### Database Initialization
1. Place your SQL scripts in `frontend/basedatos-inicio/`
2. Name your main initialization file `init.sql`
3. Restart the MariaDB container to execute:
   ```bash
   docker compose restart mariadb
   ```

### SSL Certificates
- Certificates are stored in `./acme/`
- Auto-renewal is configured via LiteSpeed's ACME client
- Custom certificates can be placed in `./acme/[domain]/`

## 📝 Usage Examples

### View Logs
```bash
# LiteSpeed access logs
docker compose logs lsws

# MariaDB logs
docker compose logs mariadb

# Follow all logs in real-time
docker compose logs -f
```

### Execute Commands
```bash
# Run Artisan commands
docker compose exec frontend php /home/ubuntu/frontend/laravel/artisan [command]

# Access MariaDB console
docker compose exec mariadb mysql -ularavel -plaravel

# Check PHP version
docker compose exec lsws php -v
```

### Management Commands
```bash
# Stop all services
docker compose down

# Restart specific service
docker compose restart lsws

# Rebuild with changes
docker compose up -d --build

# View running containers
docker compose ps
```

## 🛠️ Development Tips

### Laravel Environment
1. Configure your `.env` file:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=mariadb
   DB_PORT=3306
   DB_DATABASE=laravel
   DB_USERNAME=laravel
   DB_PASSWORD=laravel
   ```

2. Run migrations:
   ```bash
   docker compose exec frontend php /home/ubuntu/frontend/laravel/artisan migrate
   ```

### Debugging
- **PHP Errors**: Check `./logs/error.log`
- **Access Logs**: Check `./logs/access.log`
- **Database Issues**: Check `docker compose logs mariadb`

## ⚠️ Notes & Security

### Important Security Notes
1. **Default Passwords**: Change default database passwords before production use
2. **Admin Panel**: Secure the LiteSpeed admin panel (port 7080) in production
3. **SSL**: Configure proper SSL certificates for production
4. **Firewall**: Only expose necessary ports (80, 443)

### Persistence
- All application data persists in mounted volumes
- Database data is stored in `./mariadb/`
- Application code in `./frontend/laravel/`

## 🔄 Updates & Maintenance

### Updating Containers
```bash
# Pull latest images
docker compose pull

# Recreate containers
docker compose up -d
```

### Backup Database
```bash
docker compose exec mariadb mysqldump -ularavel -plaravel laravel > backup.sql
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit changes
4. Push to the branch
5. Create a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For issues and questions:
1. Check the logs: `docker compose logs`
2. Verify volumes are correctly mounted
3. Ensure ports are not in use by other services
4. Check Docker resource allocation

---

**Happy Coding!** 🚀 Built with ❤️ for the Laravel community.

---

*This README is automatically generated from the docker-compose configuration. Last updated: $(date)*
