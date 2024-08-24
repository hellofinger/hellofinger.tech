---

title: GitLab最佳实践--安装与维护
date: 2020-02-09 18:11:52
slug: gitlab-install-and-maintance
tags:
  - GitLab
  - Git
  - IAC
categories:
  - IAC

---

## 一、为什么使用Gitlab

- 杜绝因为程序员的疏忽，导致敏感信息泄露到公网。
- 开源免费。
- 满足企业的安全性与私密需求。
- 部署和维护简单。
- 可以流水线作业和项目进度协作

## 二、安装与运行

```bash
// 拉取镜像
// docker pull gitlab/gitlab-ce

// 运行镜像（因为是在本地运行，用的是本机IP地址，正式部署应该使用域名）
docker run --detach \
  --hostname 192.168.37.129 \
  --publish 443:443 --publish 80:80 --publish 2022:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  
// 修改配置
docker exec -it gitlab /bin/bash

// 编辑配置
vim /etc/gitlab/gitlab.rb

// 修改外部访问地址，局域网内可以使用IP。公网使用域名
external_url "http://192.168.37.129"

// 重启
docker restart gitlab
```

## 三、备份与恢复

### 备份

```bash
// 在主机运行备份。命令如下：
// docker exec -t <container name> gitlab-backup create

docker exec -t gitlab gitlab-backup create
// 默认的备份目录在容器目录 `/var/opt/gitlab/backups`
```

**注意 对于GitLab 12.1和更早版本，请使用 `gitlab-rake gitlab:backup:create。`** 

### 恢复

1、将备份文件拷贝到容器目录`/var/opt/gitlab/backups`。（备份和恢复的GitLab版本尽量保持一致）

2、停止相关数据连接服务

```bash
// 如果没有执行权限请加sudo
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

// 指定备份文件的时间戳前缀(多个备份文件)
gitlab-rake gitlab:backup:restore BACKUP=1570603937_2019_10_09_12.2.5
or 
gitlab-backup restore BACKUP=1570603937_2019_10_09_12.2.5

// 默认备份恢复（backups目录下只有一个备份文件时）
gitlab-rake gitlab:backup:restore

// 如果遇到权限问题，需要给备份文件增加权限
chmod 777 1570606338_2019_10_09_12.2.5_gitlab_backup.tar
```

3、重启服务

```bash
// 如果有配置文件修改，让新的配置生效
gitlab-ctl reconfigure

// 重启gitlab
gitlab-ctl start

// 检查服务
gitlab-rake gitlab:check SANITIZE=true
```

### 迁移

迁移如同备份与恢复的步骤一样，只需要将老服务器`/var/opt/gitlab/backups`目录下的备份文件拷贝到新服务器上的`/var/opt/gitlab/backups`即可(如果你没修改过默认备份目录的话)。 但是需要注意的是新服务器上的Gitlab的版本必须与创建备份时的Gitlab版本号相同。 比如新服务器安装的是最新的7.60版本的Gitlab。那么迁移之前，最好将老服务器的Gitlab 升级为7.60在进行备份。


## 四、备份数据到AWS S3

1、修改配置文件`/etc/gitlab/gitlab.rb`

```bash
gitlab_rails['backup_upload_connection'] = {
  'provider' => 'AWS',
  'region' => 'eu-west-1',
  'aws_access_key_id' => '<your-access-key-id>',
  'aws_secret_access_key' => '<your-access-key>'
  # If using an IAM Profile, don't configure aws_access_key_id & aws_secret_access_key
  # 'use_iam_profile' => true
}
gitlab_rails['backup_upload_remote_directory'] = 'my.s3.bucket'
```

2、使配置文件生效

```bash
gitlab-ctl reconfigure
```

3、执行备份命令，查看备份文件是否上传s3成功

```bash
gitlab-backup create
```

4、从s3下载备份文件到本地(必须先安装aws-cli工具)

```bash
aws s3 cp s3://finger-test/gitlab/backups/1570844249_2019_10_12_12.2.5_gitlab_backup.tar .
```

## 五、集成JIRA

```
https://docs.gitlab.com/ce/user/project/integrations/jira.html#configuring-gitlab
```

## 六、自动备份

```bash
# 使用Linux系统的Crontab
crontab -e  
# 输入相应的任务
0 2 * * * docker exec -t gitlab gitlab-backup create
```

## 参考

- https://docs.gitlab.com/omnibus/docker/
- https://docs.gitlab.com/ee/raketasks/backup_restore.html
- https://docs.gitlab.com/ce/user/project/integrations/jira.html#configuring-gitlab