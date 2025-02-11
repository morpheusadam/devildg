# devildg
the devilbox domain generator automaticly script
#!/bin/bash

# تشخیص مسیر صحیح برای devilbox
if [[ -n "$SUDO_USER" ]]; then
    # اگر اسکریپت با sudo اجرا شود، از مسیر خانه کاربر اصلی استفاده می‌کنیم
    devilbox_dir="/home/$SUDO_USER/devilbox"
else
    # در غیر این صورت، از مسیر خانه کاربر فعلی استفاده می‌کنیم
    devilbox_dir="$HOME/devilbox"
fi

# مسیر دایرکتوری پروژه‌ها
project_dir="$devilbox_dir/data/www"

# بررسی وجود دایرکتوری devilbox
if [[ ! -d "$project_dir" ]]; then
    echo "Error: Devilbox data directory not found at $project_dir."
    echo "Please ensure Devilbox is properly set up."
    exit 1
fi

# درخواست نام دامنه از کاربر
read -p "Please enter the domain name (without .local): " domain_name

# مسیر دایرکتوری پروژه جدید
new_project_dir="$project_dir/$domain_name"

# ایجاد دایرکتوری پروژه و پوشه htdocs
echo "Creating project directory at $new_project_dir..."
mkdir -p "$new_project_dir/htdocs"

# بررسی ایجاد دایرکتوری
if [[ -d "$new_project_dir/htdocs" ]]; then
    echo "Project directory created successfully: $new_project_dir/htdocs"
else
    echo "Error: Failed to create project directory at $new_project_dir/htdocs."
    echo "Please check permissions for $project_dir."
    exit 1
fi

# ایجاد فایل index.php با صفحه خوش‌آمدگویی
echo "Creating welcome page..."
cat <<EOF > "$new_project_dir/htdocs/index.php"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to $domain_name.local</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            text-align: center;
            padding: 50px;
        }
        h1 {
            color: #333;
        }
        p {
            font-size: 1.2em;
            color: #555;
        }
    </style>
</head>
<body>
    <h1>Welcome to $domain_name.local!</h1>
    <p>This page was automatically created by Devilbox.</p>
</body>
</html>
EOF

# بررسی ایجاد فایل
if [[ -f "$new_project_dir/htdocs/index.php" ]]; then
    echo "Welcome page created successfully: $new_project_dir/htdocs/index.php"
else
    echo "Error: Failed to create welcome page at $new_project_dir/htdocs/index.php."
    echo "Please check permissions for $new_project_dir/htdocs."
    exit 1
fi

# بررسی اجرا بودن کانتینر MySQL
if ! docker ps --format '{{.Names}}' | grep -q 'devilbox_mysql_1'; then
    echo "Error: MySQL container (devilbox_mysql_1) is not running."
    echo "Please start Devilbox and ensure the MySQL container is running."
    exit 1
fi

# ایجاد دیتابیس با نام مشابه دامنه
echo "Creating database..."
docker exec -it devilbox_mysql_1 mysql -uroot -e "CREATE DATABASE IF NOT EXISTS $domain_name;"

# بررسی ایجاد دیتابیس
if [[ $? -eq 0 ]]; then
    echo "Database created successfully: $domain_name"
else
    echo "Error: Failed to create database $domain_name."
    echo "Please check MySQL container logs for more details."
    exit 1
fi

# افزودن دامنه به فایل hosts در ویندوز
echo "Adding domain to the hosts file..."
if [[ -f /mnt/c/Windows/System32/drivers/etc/hosts ]]; then
    echo "127.0.0.1 $domain_name.local" | sudo tee -a /mnt/c/Windows/System32/drivers/etc/hosts > /dev/null
    if [[ $? -eq 0 ]]; then
        echo "Domain added to hosts file successfully: $domain_name.local"
    else
        echo "Error: Failed to modify the hosts file."
        echo "Please add the following line manually to your hosts file:"
        echo "127.0.0.1 $domain_name.local"
    fi
else
    echo "Error: Could not find the hosts file at /mnt/c/Windows/System32/drivers/etc/hosts."
    echo "Please add the following line manually to your hosts file:"
    echo "127.0.0.1 $domain_name.local"
fi

# پیام موفقیت
echo "Project $domain_name.local created successfully!"
echo "To view the project, visit:"
echo "http://$domain_name.local"
