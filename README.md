# Wifi-Redirection(RTL8196D)
> RTL8196D/E 기반 WebRedir 기능 구현

[![NPM Version][npm-image]][npm-url]
[![Build Status][travis-image]][travis-url]
[![Downloads Stats][npm-downloads]][npm-url]

Client의 웹 페이지 요청에 대해서 특정 URL은 허용, 나머지는 차단하여 클라이언트 웹 페이지 요청 패킷을 후킹,
해당 패킷을 공유기 자체 웹 서버로 전달하여 Redirection URL 클라이언트로 응답하는 기능 구현 

![](../header.png)

## 개발 환경 및 설정

VirtualBox(LINUX):Ubuntu-14.04.1-desktop-i386.iso-32bit

RTL8196D/E의 SDK 필요

```sh
npm install build-essential
npm install gwak
npm install bison
npm install zlib1g-dev
```
  
## 과정

busybox의 httpd 사용 체크 
공유기의 웹서버 goahead or boa 웹서버의 방화벽관련 파일
###goahead/LINUX/system/set_firewall.c에 다음 함수를 추가
```sh
#if 1   // URL Redirect - KwanDong Univ.
   char WEB_REDIR_CHAIN[]="web_redir";
#endif

#if 1   // URL Redirect - KwanDong Univ.

int set_web_redir()
{
 int val, webredirEn;
 unsigned int	lan_addr;
 char	webredirURL[32] = {0, };
 char	strLanIp[16];
 char	dest[40];
 FILE	*fp;

 system("killall -q httpd");

 RunSystemCmd(NULL_FILE, Iptables, _table, nat_table, NEW, WEB_REDIR_CHAIN, NULL_STR);
 RunSystemCmd(NULL_FILE, Iptables, _table, nat_table, INSERT, PREROUTING, jump, WEB_REDIR_CHAIN, NULL_STR);
 RunSystemCmd(NULL_FILE, Iptables, _table, nat_table, FLUSH, WEB_REDIR_CHAIN, NULL_STR);

 apmib_get(MIB_WEB_REDIR_EN, (void *)&webredirEn);

 if (webredirEn == 1)
 {
     apmib_get(MIB_WEB_REDIR_URL, (void *)&webredirURL);
       system("httpd -p 81 -h /var/web");

     fp = fopen("/var/web/index.html", "w");
     if (fp)
     {
         fprintf(fp, "<frameset>\n");
         fprintf(fp, "<frame name='CONTENT' src='http://%s' frameborder='0'>\n", webredirURL);
         fprintf(fp, "</frameset>\n");
         fclose(fp);
     }

     fp = fopen("/var/web_redir.sh", "w");
     if (fp)
     {
         apmib_get(MIB_IP_ADDR, &lan_addr);
         strcpy(strLanIp, inet_ntoa(*(struct in_addr *)&lan_addr));
         fprintf(fp, "#!/bin/sh\n");
         fprintf(fp, "iptables -A web_redir -t nat -p tcp -d %s -j ACCEPT\n", webredirURL);
         fprintf(fp, "iptables -A web_redir -t nat -p tcp -d %s --dport 80 -j DNAT --to %s:80\n", strLanIp, strLanIp);
         fprintf(fp, "iptables -A web_redir -t nat -p tcp --dport 80 -j DNAT --to %s:81\n", strLanIp);
         fclose(fp);
         chmod("/var/web_redir.sh", 0700);
         system("/var/web_redir.sh");
     }
 }
 return 0;
}
#end if

#if 1
 	apmib_get(MIB_WEB_REDIR_EN, (void *)&intVal);
 	if(intVal ==1){
     	set_web_redir();
 	}
#endif

```

###mibdef.h

```sh
#if 1   // WEB_Redirect - KwanDong Univ.
MIBDEF(unsigned char,   web_redir_en,   ,   WEB_REDIR_EN,   BYTE_T, APMIB_T, 0, 0)
MIBDEF(unsigned char,   web_redir_url, [40], WEB_REDIR_URL,  	STRING_T,   APMIB_T, 0, 0)
#endif
```

## 업데이트 내역

* 0.0.1
    * 문서 업데이트

## 정보

송기석 – https://www.rocketpunch.com/@ghgsb6200 – 이메일주소: ghgsb6200@gmail.com

XYZ 라이센스를 준수하며 ``LICENSE``에서 자세한 정보를 확인할 수 있습니다.

[https://github.com/song014/github-link](https://github.com/dbader/)

## 기여 방법

1. (<https://github.com/song014/WiFi-redirection/fork>)을 포크합니다.
2. (`git checkout -b feature/fooBar`) 명령어로 새 브랜치를 만드세요.
3. (`git commit -am 'Add some fooBar'`) 명령어로 커밋하세요.
4. (`git push origin feature/fooBar`) 명령어로 브랜치에 푸시하세요. 
5. 풀리퀘스트를 보내주세요.

<!-- Markdown link & img dfn's -->
[npm-image]: https://img.shields.io/npm/v/datadog-metrics.svg?style=flat-square
[npm-url]: https://npmjs.org/package/datadog-metrics
[npm-downloads]: https://img.shields.io/npm/dm/datadog-metrics.svg?style=flat-square
[travis-image]: https://img.shields.io/travis/dbader/node-datadog-metrics/master.svg?style=flat-square
[travis-url]: https://travis-ci.org/dbader/node-datadog-metrics
