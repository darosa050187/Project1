FROM nginx:1.23-alpine
LABEL "Project"="Vprofile" \
      "Author"="Darosa" \
      "Version"="1.0.1" \
      "environment"="Test"

RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginvproapp.conf /etc/nginx/conf.d/default.conf

# Expose ports
EXPOSE 80