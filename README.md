# .htaccess

**Сайты для проверки**, как именно редиректы обрабатывают конкретную ссылку, которую хотим проверить:
http://htaccess.mwl.be/ (htaccess tester)
http://martinmelin.se/rewrite-rule-tester/ - лучше визуально видно, какие правила сработали, но то и дело не работает

**Флаги** (NC, L, R=301, QSA и прочие) - https://httpd.apache.org/docs/current/rewrite/flags.html

Большая **коллекция готовых редиректов** для разных случаев - https://github.com/phanan/htaccess

---

Нижеуказанный код может содержать ошибки, т.к. подробно не проверялся, но отражает общие принципы работы (взято и доработано с https://gist.github.com/ScottPhillips/1721489).
Об ошибках можно сообщить в Issues или сделать Pull Request.

##301 Redirects for .htaccess (301-е редиректы через команду Redirect в .htaccess)

```apacheconf
#Redirect a single page (редирект конкретной страницы):
Redirect 301 /pagename.php http://www.domain.com/pagename.html

#Redirect an entire site (редирект всего сайта):
Redirect 301 / http://www.domain.com/

#Redirect an entire site to a sub folder (редирект всего сайта в подпапку)
Redirect 301 / http://www.domain.com/subfolder/

#Redirect a sub folder to another site (редирект подпапки на другой сайт)
Redirect 301 /subfolder http://www.domain.com/

#This will redirect any file with the .html extension to use the same filename but use the .php extension instead.
#Редирект любого файла с расширением .html на файл с таким же путем и названием, но разрешением php
RedirectMatch 301 (.*)\.html$ http://www.domain.com$1.php
```

##You can also perform 301 redirects using rewriting via .htaccess.
(Можно делать 301-е редиректы по-другому - с помощью движка RewriteEngine в .htaccess)

```apacheconf
#Redirect from old domain to new domain (редирект со старого домена на новый)
RewriteEngine on
RewriteBase /
RewriteRule (.*) http://www.newdomain.com/$1 [R=301,L]

#Redirect to www location (редирект с без-www на www-версию)
RewriteEngine on
RewriteBase /
RewriteCond %{HTTP_HOST} ^domain.com [nc]
RewriteRule ^(.*)$ http://www.domain.com/$1 [r=301,nc]

#Redirect to www location with subdirectory?
RewriteEngine on
RewriteBase /
RewriteCond %{HTTP_HOST} domain.com [NC]
RewriteRule ^(.*)$ http://www.domain.com/directory/index.html [R=301,NC]

#Redirect from old domain to new domain with full path and query string:
#Редирект со старого домена на новый с сохранением пути и параметров запроса
Options +FollowSymLinks
RewriteEngine On
RewriteRule ^(.*) http://www.newdomain.com%{REQUEST_URI} [R=302,NC]

#Redirect from old domain with subdirectory to new domain w/o subdirectory including full path and query string:
#Редирект с поддиректории старого домена на новый домен БЕЗ директории, но с сохранением остальной части пути и параметров запроса
Options +FollowSymLinks
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/subdirname/(.*)$
RewriteRule ^(.*) http://www.katcode.com/%1 [R=302,NC]

#Rewrite and redirect URLs with query parameters (files placed in root directory)
#Редирект ссылки в корневой папке с конкретным параметром на папку
#УДАЛЕНИЕ параметров - за счет добавления знака вопроса в целевой ссылке (см. пример)
#Original URL:
#http://www.example.com/index.php?id=1
#Desired destination URL:
#http://www.example.com/path-to-new-location/

#.htaccess syntax:
RewriteEngine on
RewriteCond %{QUERY_STRING} id=1
RewriteRule ^index\.php$ /path-to-new-location/? [L,R=301]

#Redirect URLs with query parameters (files placed in subdirectory)
#Редирект ссылок в подпапке в корне сайта с конкретным параметром, параметры после редиректа обрезаются знаком вопроса
#Original URL:
#http://www.example.com/sub-dir/index.php?id=1
#Desired destination URL:
#http://www.example.com/path-to-new-location/

#.htaccess syntax:
RewriteEngine on
RewriteCond %{QUERY_STRING} id=1
RewriteRule ^sub-dir/index\.php$ /path-to-new-location/? [L,R=301]

#Redirect one clean URL to a new clean URL
#Редирект конкретной ссылки БЕЗ параметров (за счет знака вопроса) на другую ссылку
#Original URL:
#http://www.example.com/old-page/
#Desired destination URL:
#http://www.example.com/new-page/

#.htaccess syntax:
RewriteEngine On
RewriteRule ^old-page/?$ $1/new-page$2 [R=301,L]

#Rewrite and redirect URLs with query parameter to directory based structure, retaining query string in URL root level
#Редирект ссылок с разными id в параметре на папки с соответствующими id, С СОХРАНЕНИЕМ других параметров (за счет флага QSA)
#Original URL:
#http://www.example.com/index.php?id=100&param=123
#Desired destination URL:
#http://www.example.com/100/?param=123

#.htaccess syntax:
RewriteEngine On
RewriteRule ^index.php?id=([/d]+) /$1/ [QSA]

#Rewrite URLs with query parameter to directory based structure, retaining query string parameter in URL subdirectory
#Редирект с главной страницы на папку, соответствующую значению параметра, с сохранением других параметров за счет флага QSA
#Original URL:
#http://www.example.com/index.php?category=fish&param=123
#Desired destination URL:
#http://www.example.com/category/fish/?param=123

#.htaccess syntax:
RewriteEngine On
RewriteRule ^index.php?category=([^/d]+)$ /category/$1/ [L,QSA]

#Domain change – redirect all incoming request from old to new domain (retain path)
#Редирект сайта с одного домена на другой
RewriteEngine on
RewriteCond %{HTTP_HOST} ^example-old\.com$ [NC]
RewriteRule ^(.*)$ http://www.example-new.com/$1 [R=301,L]

#If you do not want to pass the path in the request to the new domain, change the last row to:
#То же самое, но с редиректом на главную страницу нового домена
RewriteRule ^(.*)$ http://www.example-new.com/ [R=301,L]

#From blog.oldsite.com -> www.somewhere.com/blog/
#retains path and query, and eliminates xtra blog path if domain is blog.oldsite.com/blog/
#Редирект с поддомена на папку (blog.oldsite.com -> www.somewhere.com/blog/) с сохранением путей и параметров запроса
Options +FollowSymLinks
RewriteEngine On
RewriteCond %{REQUEST_URI}/ blog
RewriteRule ^(.*) http://www.somewhere.com/%{REQUEST_URI} [R=302,NC]
RewriteRule ^(.*) http://www.somewhere.com/blog/%{REQUEST_URI} [R=302,NC]
```
