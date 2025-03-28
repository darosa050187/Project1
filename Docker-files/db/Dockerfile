FROM mysql:8.0.33
LABEL "Project"="Vprofile" \
      "Author"="Darosa" \
      "Version"="1.0.1" \
      "environment"="Test"
# Set working directory
WORKDIR /docker-entrypoint-initdb.d
ENV MYSQL_ROOT_PASSWORD="vprodbpass"
ENV MYSQL_DATABASE="accounts"
COPY db_backup.sql .
# Expose MySQL port
EXPOSE 3306