# docker
#Base Image ## sudo chmod 666 /var/run/docker.sock
#docker build -t satya/ssh:0.0.1 . -f ssh-dockerfile
#docker network create --driver bridge my-network
#docker run -d -p 32:22 --name ssh-svr --network some-network satya/ssh:0.0.1
#sudo ufw allow proto tcp from any to any port 22
#sudo ufw allow proto tcp from any to any port 32
#sudo ufw enable
#sudo ufw status
#docker port ssh-svr 22
#ssh root@127.0.0.1 -p 32
#mysql -h mysql-server -uguest -p
#sudo docker container stop $(sudo docker container ls -aq) && sudo docker system prune -af --volumes
