# gitlab-install

container üzerinde kurmak için.

```mkdir gitlab
docker run -d --hostname gitlab.dev-ops.expert \
-p 443:443 -p 80:80 -p 2222:22 \
--name gitlab \
--restart unless-stopped \
-v /root/gitlab/config:/etc/gitlab \
-v /root/gitlab/logs:/var/log/gitlab \
-v /root/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```

root password için container içinde /etc/gitlab/initial_root_password dan alırız.

runner için.

```
mkdir gitlab-runner
docker run -d --name gitlab-runner --restart always \
    -v /root/gitlab-runner/config:/etc/gitlab-runner \
    -v /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner:v14.7.0
``` 

runner kurulumdan sonra gitlab-runner register diyerek gitlab hostname i ve gitlab arayüzünden aldığımız token ile register oluyoruz.

ardından toml dosyasında priviled yapmamız gerekiyor.

```
sed -i 's,privileged = false,privileged = true,g' /etc/gitlab-runner/config.toml
```

örnek gitlab-ci yaml için


```
image: alpine
stages:
    - test
test:
    image: docker
    services: 
        - name: docker:20-dind
          alias: docker
          command: ["--tls=false"]
    stage: test
    script:
        - sleep 5
        - echo "hello gitlab-ci"
    tags: 
        - docker-runner
