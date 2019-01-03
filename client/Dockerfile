FROM node:carbon

COPY ./package*.json /client/
COPY ./docker/entrypoint.sh /entrypoint/


RUN ["chmod", "+x", "entrypoint/entrypoint.sh"]

# Copy or mount node app here
# WORKDIR /client/

# EXPOSE 8080

ENTRYPOINT ["/entrypoint/entrypoint.sh"]

# CMD ["npm", "start"]



