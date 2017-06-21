---
layout: post
title: GitLab備份筆記
comments: true
---


<img src="https://cdn.xebialabs.com/assets/files/plugins/gitlab.jpg" alt="Gitlab" height="100">
<br>
GitLab CE是開源應用程式，它擁有我最喜歡的開源專案特性。**就是免費**～～～喔還有強大完整，因此是當初在找內部使用的git平台方案的首選。在自己電腦最早也有用<a href="https://www.vagrantup.com/" target="_blank">Vagrant</a>架設虛擬機器安裝GitLab來丟一些個人專案，隨著時間推移改用Docker時就勢必得<a href="https://docs.gitlab.com/omnibus/settings/backups.html#creating-an-application-backup" target="_blank">備份</a>後扔進Docker中<a href="https://docs.gitlab.com/ce/raketasks/backup_restore.html#restore-for-omnibus-installations" target="_blank">還原</a>。這邊簡單描述大概步驟，跟遇到的坑。

## Creating backups

單純匯出備份其實很簡單，預設的備份資料夾設定在```/etc/gitlab/gitlab.rb```文件中:

{% highlight Ruby %}
# 預設備份資料夾位置
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
# 厲害的還有備份到雲端的設定
# gitlab_rails['backup_upload_connection'] = {
#   'provider' => 'AWS',
#   'region' => 'eu-west-1',
#   'aws_access_key_id' => 'AKIAKIAKI',
#   'aws_secret_access_key' => 'secret123'
# }
# gitlab_rails['backup_upload_remote_directory'] = 'my.s3.bucket'
{% endhighlight %}
簡單指令就能創建備份檔
{% highlight Shell %}
sudo gitlab-rake gitlab:backup:create
{% endhighlight %}

## Restore
還原也很簡單，將備份檔扔回```/var/opt/gitlab/backups```文件夾中
{% highlight Shell %}
# 停止數據庫部分功能
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq
# 從檔案 1498061316_2017_06_21_9.2.2_gitlab_backup.tar 還原
sudo gitlab-rake gitlab:backup:restore BACKUP=1498061316_2017_06_21_9.2.2
# 重啟跟檢查
sudo gitlab-ctl start
sudo gitlab-rake gitlab:check SANITIZE=true
{% endhighlight %}
天啊好簡單，但是我還是踩到坑了。
> If there is a GitLab version mismatch between your backup tar file and the installed version of GitLab, the restore command will abort with an error. Install the correct GitLab version and try again.

在備份跟還原前就知道有警告說版本不同可能會無法還原，不要不信邪，果然GitLab真的就不給你還原。還好升級GitLab很簡單，<a href="https://about.gitlab.com/update/" target="_blank">升級</a>後再匯出備份一次重來吧！

{% highlight Shell %}
sudo apt-get update 
sudo apt-get install gitlab-ce
{% endhighlight %}
