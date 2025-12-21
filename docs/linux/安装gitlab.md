
https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el9/


vim /etc/gitlab/gitlab.rb

external_url 更改为自己的发布地址

在 /etc/gitlab/gitlab.rb 里真正决定“数据存到哪”的只有一行：
git_data_dirs({ "default" => { "path" => "/var/opt/gitlab/git-data" } })
默认路径就是 /var/opt/gitlab/git-data（所有仓库的 .git 目录都在它下面）。
如果你想改到别的磁盘，只要把这行改成你自己的目录，然后 gitlab-ctl reconfigure 即可。

prometheus['enable'] = false

gitlab-ctl reconfigure

