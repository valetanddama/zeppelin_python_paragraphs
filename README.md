# Python: work with zeppelin api for paragraphs
Python v2.7

```python
%python
import smtplib
import urllib2
import json
from urllib2 import HTTPError
from email.mime.text import MIMEText
import cookielib

class ParagraphMonitoring:

    def __init__(self):
        self.urlAuth = 'https://<host_of_zeppelin>/api/login'
        self.urlParagraph = 'https://<host_of_zeppelin>/api/notebook/<notebook_id>/paragraph/<paragraph_id>'
        self.headers = {
            'content-type': 'application/json',
            'Cookie': ''
        }

    def auth(self):
        try:
            payload = "userName=<username>&password=<password>"
            self.headers['content-type'] = "application/x-www-form-urlencoded"

            request = urllib2.Request(self.urlAuth, data=payload, headers=self.headers)
            request.get_method = lambda: 'POST'

            cookies = cookielib.CookieJar()
            opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookies), urllib2.HTTPHandler())

            opener.open(request)
            self.set_cookies(cookies)

        except HTTPError as e:
            self.send_email('Script is failed')

    def check_data(self):
        try:
            self.auth()

            request = urllib2.Request(self.urlParagraph, data=None, headers=self.headers)
            request.get_method = lambda: 'GET'

            response = urllib2.urlopen(request)
            data = json.load(response)
            
            #your data in data['body']['results']['msg'], for example: data from paragraph is_error: 0
            if data['body']['results']['msg'][0]['data'] == "is_error\n0\n":
                self.send_email('Error')

        except HTTPError as e:
            self.send_email('Script is failed')

    def send_email(self, text):
        from_email = '<from_email>'
        to_email = '<to_email>'

        message = MIMEText(text)
        message['From'] = from_email
        message['To'] = to_email
        message['Subject'] = 'Paragraph monitoring exception'

        s = smtplib.SMTP('<smtp_server>', 25)

        try:
            s.sendmail(from_email, [<array_of_emails_for_send>], message.as_string())
        except Exception as e:
            print 'Something went wrong...'

        s.quit()

    def set_cookies(self, cookies):
        for cookie in cookies:
            if cookie.name == 'JSESSIONID':
                self.headers['Cookie'] = cookie.name + '=' + cookie.value


paragraphMonitoring = ParagraphMonitoring()
paragraphMonitoring.check_data()
```
