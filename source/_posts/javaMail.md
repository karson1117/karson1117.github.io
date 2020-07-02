---
title: javaMail发送邮件
date: 2017-05-25 14:39:47
tags:
	- JavaMail
categories:
	- 编程
	- Java
---
>使用163邮箱作为邮件测试时遇到身份验证不通过的问题,在此略做记录..

<!--more-->	
### 部分代码
{% codeblock %}
	/**
	 * 发送邮件
	 * 
	 * @param toAddress		: 收件人邮箱
	 * @param mailSubject	: 邮件主题
	 * @param mailBody		: 邮件正文
	 * @param mailBodyIsHtml: 邮件正文格式,true:HTML格式;false:文本格式
	 * //@param inLineFile	: 内嵌文件
	 * @param attachments	: 附件
	 */
	public static boolean sendMail (String toAddress, String mailSubject, String mailBody, 
			boolean mailBodyIsHtml, File[] attachments){
        try {
			// 创建邮件发送类 JavaMailSender (用于发送多元化邮件，包括附件，图片，html 等    )
        	JavaMailSenderImpl mailSender = new JavaMailSenderImpl();
        	mailSender.setHost(host); 			// 设置邮件服务主机    
        	mailSender.setUsername(username); 	// 发送者邮箱的用户名
        	mailSender.setPassword(password); 	// 发送者邮箱的密码
        	
			//配置文件，用于实例化java.mail.session    
			Properties pro = new Properties();
			pro.put("mail.smtp.auth", "true");		
			pro.put("mail.smtp.socketFactory.port", port);
			pro.put("mail.smtp.socketFactory.fallback", "false");
			pro.put("mail.smtp.starttls.enable", "true");
			mailSender.setJavaMailProperties(pro);
	
			//创建多元化邮件 (创建 mimeMessage 帮助类，用于封装信息至 mimeMessage)
			MimeMessage mimeMessage = mailSender.createMimeMessage();
			MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, ArrayUtils.isNotEmpty(attachments), "UTF-8");
			
			helper.setFrom(sendFrom, sendNick);
			helper.setTo(toAddress);
	
			helper.setSubject(mailSubject);
			helper.setText(mailBody, mailBodyIsHtml); 
			
			// 添加内嵌文件，第1个参数为cid标识这个文件,第2个参数为资源
			//helper.addInline(MimeUtility.encodeText(inLineFile.getName()), inLineFile);	
			
			// 添加附件    
			if (ArrayUtils.isNotEmpty(attachments)) {
				for (File file : attachments) {
					helper.addAttachment(MimeUtility.encodeText(file.getName()), file);
				}
			}
			
			mailSender.send(mimeMessage);
			return true;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return false;
	}
{% endcodeblock %}

### 运行报错：
>“Authentication failed; nested exception is javax.mail.AuthenticationFailedException: 550 User has no permission.....”

### 解决
网易163邮箱 "设置--POP3/SMTP/IMAP" 中 "客户端授权" 未开启。
开启授权码后，它将代替邮箱密码在客户端使用。
![](/img/articleImg/163sqm.png)




