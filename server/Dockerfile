FROM node:21-alpine3.18 

WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["npm", "start"]

