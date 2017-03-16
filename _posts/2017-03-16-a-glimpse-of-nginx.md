---
layout: post
title:  "Nginx��̽(A Glimpse of Nginx)"
date:   2017-03-16 23:06:19
categories: Java
---
ǰ��ʱ���ڰ��ֹ�˾��΢�Ź��ںţ�ʹ�õ���Nginx�������������Nginx����Ǹ��ö�����
>**��Ϊʲôʹ��Nginx��**

���ǵ�΢�Ź��ںſ�����ʹ��ǰ��˷���ļܹ������ʹ��java��WebService, ǰ�˾�����html5 + jQuery���������˵��web ������ѡ��ļ����׶Ρ�

**Phase 1: tomcat + ecstatic server��΢�Ź��ڲ��Ժţ�**
Tomcat��������java web����ѡweb����������������ʹ��tomcat��web��������������ǰ��˷��룬ǰ��Ҳ��Ҫ������������ʹ�õ���node �� ecstatic server��ʹ��npm���߿��Ի�á�����΢��Ҫ�󿪷�����������΢����Ϣ���¼��Ľӿ�URLֻ����http://��https://��ͷ��Ҳ����˵ֻ֧��80�Լ�443�˿ڡ�����tomcat�Ͳ���ʹ�����ǳ��õ�8080�˿ڣ���Ҫ�ĳ�80�˿ڣ���server.xml���޸ģ���ecstatic serverʹ��9000�˿ڣ��Ϳ�����ǰ��˷ֱ𿪷����ֿ�����

**Phase 2: tomcat**
��Phase 1������ʹ�õ���΢�Ų��Ժţ�Phase 2����ʹ�ùٷ���֤�Ĺ��ںš����ǣ���Ǩ�ƵĹ����оͷ�����һЩ���⡣�������ǵĹ��ں�ʹ����JSSDK����Ҫ���ð�ȫ��������Phase 1������д�ģ�XXX.XXX.XXX.XXX:9000�����ǵ���Phase 2����д���ǲ�������ˣ���֧��ʹ��ip�Լ��˿ڣ�ֻ������ͨ��ICP������֤���������������ﲻ����ʹ��9000�˿��ˡ�����Phase 2ǰ��server���ʹ����tomcat������ǰ��˶�ʹ��tomcat��Ϊserver��������������server.xml�����һ�У�
<Context path="/" docBase="C:/ YOUR-FRONT-END-PATH " reloadable="true" debug="0" crossContext="true"/>

**Phase 3: Nginx + tomcat**
Phase 2�ƺ��Ѿ��󹦸���ˣ�����������Ȼ�кܴ�����⡣��������޸���ǰ�˴��룬��������tomcat���ܹ���Ч����Ȼ������̸ʲôǰ��˷��룬�����Ͳ�̫���ˡ�����ô���ܱ���tomcat����η�����أ���Ȼǰ��˿��Բ��𵽲�ͬ��server������Ȼ���ụ��Ӱ�죬������ֻ��һ̨server������£������ſ��������أ����ʱ��Nginx�ͺ�ճ����ˡ�����˼����ǿͻ���ֻ��Nginx�򽻵�����Nginx��Ϊ����tomcat�Ĵ���tomcat������java����ʹ��8080�˿ڣ���ǰ�˾�ֱ�Ӱ�Nginx��Ϊ��̬��Դ�����������������Nginxʹ��80�˿ڡ�������Nginx�����ļ��Ĺؼ�Ƭ�Σ�

```
location / {
			proxy_pass http://127.0.0.1:8080;
}
location ~ \.(html|js|css|png|gif)$ {    
			root C:/YOUR-FRONT-END-PATH;
			expires 24h;
}
```

����ֱ���java����Լ�ǰ�˵����á�ע��������һ���ӣ�http://127.0.0.1:8080��Ҫд��localhost:8080, �����ᵼ��ǰ���ļ����������������ܼ��ء���������Ǻ��˲�ǳ����
���ˣ��Ѿ�������Ҷ�Nginx�ĵ�һ�νӴ���
��ĩżȻ����꿴��һ������Nginx���顶�������Nginx��,����һ���磬ȼ�����Ҷ����Ķ�NginxԴ������顣Github������Nginx��Դ�룺https://github.com/nginx/nginx�����ⷢ���Ա���tengine�Ŷ�Ҳ�кܺõĽ���NginxԴ������ϣ�http://tengine.taobao.org/book/index.html��׼����һ�׶��мƻ��Ŀ�ʼ���������Ķ�NginxԴ�롣


