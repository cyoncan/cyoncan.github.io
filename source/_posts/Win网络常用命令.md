---
title: Windows���糣������
date: 2017-03-23 20:56:21
categories:
- Windows
tags:
- cmd
---

CMD����������

```powershell
arp -a �г������������л�Ծ��IP��ַ
arp -a �ӶԷ�IP�ǲ�Է���MAC��ַ
arp -s ��ip + mac����mac��ip��ַ
arp -d ��ip + mac�����mac��ip��ַ

net view ����> ��ѯͬһ���ڻ����б�
net view /domain ����> ��ѯ���б�
net view /domain:domainname ���C> �鿴workgroup���м�����б�

ipconfig /all ����> ��ѯ����IP�Σ��������
ipconfig /release
ipconfig /renew ���»�ȡIp��ַ

telnet ip �˿ںţ������ܷ������Զ�������˿� nbtstat -a �ӶԷ�IP��Է���������
tracert ������ �õ�IP��ַ

netstat -a -n
netstat -an | find ��3389��
netstat -a�鿴������Щ�˿�
netstat -n�鿴�˿ڵ������������
netstat -v�鿴���ڽ��еĹ���
netstat -p tcp/ip�鿴ĳЭ��ʹ�����
netstat -s �鿴����ʹ�õ�����Э��ʹ�����

nbtstat -n ��ȡNetBIOS
nslookup ���� ��ѯ������Ӧ��ip
```