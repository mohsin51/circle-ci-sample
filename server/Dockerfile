FROM node:carbon

# COPY ./package*.json /server/
COPY ./docker/entrypoint.sh /entrypoint/


RUN ["chmod", "+x", "entrypoint/entrypoint.sh"]

# Copy or mount node app here
# WORKDIR /server/

# EXPOSE 8081

ENTRYPOINT ["/entrypoint/entrypoint.sh"]

# CMD ["npm", "start"]

