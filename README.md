# AlKarma TV IPTV Admin Panel

A full-stack IPTV admin panel for managing devices, channels, and news for AlKarma TV. This application allows administrators to manage devices, their permissions, channels, and news content, with appropriate APIs for client devices.

## Tech Stack

### Frontend
- **React.js** - Frontend library for building user interfaces
- **React Router** - For client-side routing
- **React Bootstrap** - UI component library
- **Formik & Yup** - Form handling and validation
- **Axios** - HTTP client for API requests
- **React Icons** - Icon library
- **React-Toastify** - For displaying notification messages

### Backend
- **Node.js** - JavaScript runtime
- **Express** - Web framework for Node.js
- **SQLite3** - Lightweight, serverless database
- **Multer** - For handling file uploads (channel logos)
- **CORS** - For cross-origin resource sharing

## Features

- **Dashboard** - Overview of system stats, expiring devices, and recent actions
- **Devices Management** - CRUD operations for devices, activation codes, and permissions
- **Channels Management** - CRUD operations for channels with logo uploads
- **News Management** - CRUD operations for news articles
- **Client APIs** - APIs for device registration, activation, and content delivery

## Project Structure

```
AlKarma-TV/
├── client/                  # Frontend React application
│   ├── public/              # Static files
│   └── src/                 # React source code
│       ├── components/      # Reusable components
│       ├── pages/           # Page components
│       ├── services/        # API services
│       └── utils/           # Utility functions
├── server/                  # Backend Node.js application
│   ├── controllers/         # API controllers (future implementation)
│   ├── models/              # Database models (future implementation)
│   ├── routes/              # API routes
│   ├── uploads/             # Folder for channel logos
│   ├── database.sqlite      # SQLite database file
│   └── index.js             # Server entry point
└── README.md                # Project documentation
```

## Installation and Development

### Prerequisites
- Node.js (v16.x or higher)
- npm (v8.x or higher)
- PM2 (Process Manager) for running in production

### Development Setup (Windows)

#### Option 1: Single Command Startup (Recommended)

1. **Clone the repository**
   ```bash
   git clone https://your-repository-url/AlKarma-TV.git
   cd AlKarma-TV
   ```

2. **Install dependencies**
   ```bash
   npm install -g pm2
   cd server && npm install && cd ../client && npm install && cd ..
   ```

3. **Start both server and client with PM2**
   ```bash
   pm2 start ecosystem.config.js
   ```
   This will:
   - Start the server at http://localhost:5000 
   - Start the client at http://localhost:3000
   - Monitor both processes

4. **View logs**
   ```bash
   pm2 logs
   ```

5. **Stop all processes**
   ```bash
   pm2 stop all
   ```

#### Option 2: Separate Terminal Startup

1. **Clone the repository**
   ```bash
   git clone https://your-repository-url/AlKarma-TV.git
   cd AlKarma-TV
   ```

2. **Install backend dependencies**
   ```bash
   cd server
   npm install
   ```

3. **Start the backend server**
   ```bash
   npm run dev
   ```
   This will start the server at http://localhost:5000

4. **Install frontend dependencies (in a separate terminal)**
   ```bash
   cd AlKarma-TV/client
   npm install
   ```

5. **Start the frontend development server**
   ```bash
   npm start
   ```
   This will start the client at http://localhost:3000

### Building for Production

1. **Build the React frontend**
   ```bash
   cd client
   npm run build
   ```

2. **Prepare the server for production**
   ```bash
   cd ../server
   npm install --production
   ```

## Deployment (Ubuntu Server)

### Prerequisites
- Ubuntu Server (18.04 LTS or higher)
- Node.js (v16.x or higher)
- npm (v8.x or higher)
- PM2 (Process Manager for Node.js applications)

### Production Setup

1. **Install Node.js and npm**
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```

2. **Install PM2 globally**
   ```bash
   sudo npm install pm2 -g
   ```

3. **Clone the repository on the server**
   ```bash
   git clone https://your-repository-url/AlKarma-TV.git
   cd AlKarma-TV
   ```

4. **Install dependencies and build for production**
   ```bash
   # Install server dependencies
   cd server
   npm install --production
   cd ..
   
   # Install and build client
   cd client
   npm install
   npm run build
   cd ..
   ```

5. **Create production ecosystem file**
   Create a file named `ecosystem.production.config.js`:
   ```javascript
   module.exports = {
     apps: [
       {
         name: 'alkarma-tv-server',
         script: 'server/index.js',
         env_production: {
           NODE_ENV: 'production',
           PORT: 5000
         },
         instances: 1,
         autorestart: true,
         watch: false,
         max_memory_restart: '1G'
       }
     ]
   };
   ```

6. **Setup Nginx to serve static files and proxy API requests**

   Install Nginx:
   ```bash
   sudo apt install nginx
   ```

   Create a new Nginx site configuration:
   ```bash
   sudo nano /etc/nginx/sites-available/alkarma-tv
   ```

   Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name your_domain_or_IP;

       # Serve React static files
       location / {
           root /path/to/AlKarma-TV/client/build;
           try_files $uri /index.html;
           expires 1d;
           add_header Cache-Control "public, max-age=86400";
       }

       # Proxy API requests
       location /api {
           proxy_pass http://localhost:5000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }

       # Serve uploaded files
       location /uploads {
           proxy_pass http://localhost:5000;
           sendfile on;
           tcp_nopush on;
           tcp_nodelay on;
       }
   }
   ```

   Enable the site and restart Nginx:
   ```bash
   sudo ln -s /etc/nginx/sites-available/alkarma-tv /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl restart nginx
   ```

7. **Start the server with PM2**
   ```bash
   pm2 start ecosystem.production.config.js --env production
   ```

8. **Setup PM2 to start on boot**
   ```bash
   pm2 startup
   pm2 save
   ```

9. **Setup automatic database backup (optional but recommended)**
   Create a backup script:
   ```bash
   mkdir -p /path/to/AlKarma-TV/backups
   nano /path/to/AlKarma-TV/backup.sh
   ```

   Add the following content:
   ```bash
   #!/bin/bash
   DATE=$(date +%Y%m%d_%H%M%S)
   cp /path/to/AlKarma-TV/server/database.sqlite /path/to/AlKarma-TV/backups/database_$DATE.sqlite
   # Keep only the 10 most recent backups
   ls -tp /path/to/AlKarma-TV/backups/ | grep -v '/$' | tail -n +11 | xargs -I {} rm -- /path/to/AlKarma-TV/backups/{}
   ```

   Make the script executable:
   ```bash
   chmod +x /path/to/AlKarma-TV/backup.sh
   ```

   Add it to crontab to run daily:
   ```bash
   (crontab -l 2>/dev/null; echo "0 0 * * * /path/to/AlKarma-TV/backup.sh") | crontab -
   ```

### Maintenance and Monitoring

- **View logs**:
  ```bash
  pm2 logs alkarma-tv-server
  ```

- **Restart the application**:
  ```bash
  pm2 restart alkarma-tv-server
  ```

- **Monitor the application**:
  ```bash
  pm2 monit
  ```

- **Update the application**:
  ```bash
  cd /path/to/AlKarma-TV
  git pull
  cd client
  npm install
  npm run build
  cd ../server
  npm install --production
  pm2 restart alkarma-tv-server
  ```

## API Documentation

### Client APIs

#### Check Device Registration
- **URL**: `/api/client/check-device`
- **Method**: `POST`
- **Body**: 
  ```json
  {
    "duid": "DEVICE_UNIQUE_ID"
  }
  ```
- **Response**: 
  - If device is active: Returns device info, allowed channels, and news
  - If device is expired: Returns device info, FTA/Local channels, and news
  - If device is disabled: Returns device info and a message
  - If device is not found: Returns error message

#### Register New Device
- **URL**: `/api/client/register-device`
- **Method**: `POST`
- **Body**: 
  ```json
  {
    "device_name": "Device Owner Name"
  }
  ```
- **Response**: 
  - Returns the newly created device info with DUID and activation code

#### Activate Device
- **URL**: `/api/client/activate-device`
- **Method**: `POST`
- **Body**: 
  ```json
  {
    "duid": "DEVICE_UNIQUE_ID",
    "activation_code": "1234"
  }
  ```
- **Response**: 
  - If successful: Returns device info with active status
  - If invalid code: Returns error message

### Admin Panel APIs

The admin panel APIs include CRUD operations for devices, channels, and news. These are accessible through the admin panel interface.

## License

This project is proprietary and confidential. Unauthorized copying, modification, distribution, or use of this software is strictly prohibited.

## Support

For support and inquiries, please contact the project maintainer.
