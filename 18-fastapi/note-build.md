# This script builds a custom Odoo 18 Docker image with additional dependencies.
docker login
docker build -t arisnew/odoo18-custom . 
docker push arisnew/odoo18-custom:latest