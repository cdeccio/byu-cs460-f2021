# Hands-On with the Application Layer

The objectives of this assignment are to gain hands-on experience with
application-layer protocols such as HTTP, DNS, and SMTP.


# Part 1 - HTTP Cookies and Conditional GET

## Getting Started

 1. Backup and Modify `/etc/hosts`.

    First, backup your `/etc/hosts` by creating a copy of it at
    `/etc/hosts.bak`:
 
    ```
    sudo cp -pr /etc/hosts{,.bak}
    ```

    Now open `/etc/hosts` with root privileges, e.g.:

    ```
    sudo -e /etc/hosts
    ```

    Modify the first line so it looks like this:

    ```
    127.0.0.1	localhost bar.com foo.bar.com bar.net
    ```

    Then close and save the file.  That makes it so that applications on your
    virtual machine (VM) resolve `bar.com`, `foo.bar.com`, and `bar.net` to
    127.0.0.1, i.e., the loopback address on your system.  Thus, when your
    browser attempts to retrieve the Web page at `bar.com` (or any of the
    others), it will connect the Web server you will start on your VM
    (listening on 127.0.0.1), rather than some external host.


 2. Begin Packet Capture.  Open Wireshark as the root user:

    ```
    sudo wireshark
    ```

    Enter `port 8000` into the capture filter field field.  Then double-click
    "Loopback: lo" to begin capturing on the loopback interface.


 3. Start Web Server.

    In another terminal window or tab, start a Python-based Web server with
    CGI-enabled:

    ```bash
    $ python3 -m http.server --cgi
    ```

 4. Open Web Browser.  Open firefox, and clear all of its cache.

 5. Issue Web Requests.  From the newly-opened Web browser window, issue the
    following requests, one after the other:

    1. `http://bar.com:8000/cgi-bin/test.cgi`
    2. `http://bar.com:8000/cgi-bin/test.cgi` (a second time - you might need to click the reload button)
    3. `http://foo.bar.com:8000/cgi-bin/test.cgi`
    4. `http://bar.net:8000/cgi-bin/test.cgi`
    5. `http://bar.com:8000/test.txt`
    6. `http://bar.com:8000/test.txt` (a second time - you might need to click the reload button)

 6. "Update" `test.txt`, and re-request it.  Run the following command:

    `$ touch test.txt`

    Then re-request the following URL:

    7. `http://bar.com:8000/test.txt` (a third time - you might need to click the reload button)

 7. Follow TCP Streams.  For each of the URLs requested in #5 and #6, open the
    corresponding TCP stream by following the instructions below:

    1. Find the frame that includes the request, e.g., "GET /cgi-bin/test.cgi",
       right-click on that frame and select "Follow", then "TCP Stream".  Leave
       open the "Follow TCP Stream" window that is created.
    2. Go back to the Wireshark capture window, and clear the display filter
       (should say "tcp.stream eq 1" or the like).  Then go back to #1 for the
       next URL.

## Questions

Consider each of the HTTP requests issued above when answering the following
questions.  Use the TCP stream windows to help you understand and answer the
questions.
 
 1. In request #1, was a cookie sent by the client?  Why or why
    not?
 2. In request #2, was a cookie sent by the client?  Why or why
    not?
 3. In request #3, was a cookie sent by the client?  Why or why
    not?
 4. In request #4, was a cookie sent by the client?  Why or why
    not?
 5. In request #5, was a cookie sent by the client?  Why or why
    not?
 6. What was the response code associated with request #5?
 7. What was the response code associated with request #6?  Why was it
    different than that of request #5?  Be brief but specific.
 8. What was the response code associated with request #7?  Why was it
    different than that of request #6?  Be brief but specific.


## Cleanup

 1. Revert to your backup of `/etc/hosts`:

    ```
    sudo mv /etc/hosts{.bak,}
    ```

 2. Close Wireshark.

 3. Close Firefox


# Part 2 - TCP Fast Open

This part is an exercise to help you understand TCP Fast Open (TFO).  The
script `tfo_echo.py` can be run both as an echo client and an echo server,
depending on the presence of the `-l` option.  When the script is run with the
`-f` option (whther client or server), TFO is used.


## Getting Started

 1. Start cougarnet.  File `h2-s1.cfg` contains a configuration file that
    describes a network with two hosts, `a` and `b`, connected to switch `s1`.

    Run the following command to create and start the network:

    ```bash
    cougarnet --display -w s1 h2-s1.cfg
    ```

 2. Begin Packet Capture.  Go to the open Wireshark window, click the "Capture
    Options" button (the gear icon).  Select the `s1-a` interface for packet
    capture.

 3. Start and interact with an echo server. Run the following on host `b` to
    start the echo server:

    ```bash
    b$ python3 tfo_echo.py -l 5599
    ```

    On host `a`, running the following to run the client:

    ```bash
    a$ python3 tfo_echo.py 10.0.0.2 5599 foobar
    ```


## Questions (1)

Answer the following questions about the packet capture:

 1. What were the relative sequence number and the segment length (i.e., the
    TCP payload) associated with the SYN packet?

 2. What was the relative acknowledgement number associated with the SYNACK packet?

 3. How many RTTs did it take for the string "echoed" by the server to be
    received by the client, including connection establishment?  (Note: don't
    actually add up the time; just think about and perhaps draw out the back
    and forth interactions between client and server.)

 4. Was there a TFO option in the TCP header?  If so, was there a cookie, and
    what was its value?


## Enabling and Priming TFO

Use `Ctrl`-`c` on host `b` to interrupt the running echo server.  Then run
the following on both host `a` and host `b`:

```bash
$ sudo sysctl net.ipv4.tcp_fastopen=3
```

Depending on the value passed to the `net.ipv4.tcp_fastopen` value, TFO might
be enabled for only when the host is acting as a TCP client, only when the host
is acting as a TCP server, or both.  The value 3 enables TFO from both a client
perspective _and_ a server perspective.

Restart the server on host `b` with the following command (note the presence of
the `-f` option):

```bash
b$ python3 tfo_echo.py -f -l 5599
```

Now run the client again on host `a` with the following command (note the
presence of the `-f` option):

```bash
a$ python3 tfo_echo.py -f 10.0.0.2 5599 foobar
```


## Questions (2)

For questions 5 - 8, answer the same questions as 1 - 4, but for the most
recent test.


## Using TFO

Finally, restart the server on host `b` with the following command:

```bash
b$ python3 tfo_echo.py -f -l 5599
```

Then run the following again:

```bash
a$ python3 tfo_echo.py -f 10.0.0.2 5599 foobar
```


## Questions (3)

For questions 9 - 12, answer the same questions as 1 - 4, but for the most
recent test.

 13. Looking `tfo_echo.py`, what key differences are involved in programming a
     TFO connection (vs. a non-TFO TCP connection) from the perspective of the
     _client_.


# Part 3 - SMTP

This part is an exercise to help you understand SMTP.

## Getting Started

 1. Install swaks (Swiss Army Knife SMTP). Run the following to install swaks:

    ```
    sudo apt install swaks
    ```

 2. Start cougarnet.  File `h2-s1.cfg` contains a configuration file that
    describes a network with two hosts, `a` and `b`, connected to switch `s1`.

    Run the following command to create and start the network:

    ```bash
    cougarnet --display -w s1 h2-s1.cfg
    ```

 3. Begin Packet Capture.  Go to the open Wireshark window, click the "Capture
    Options" button (the gear icon).  Select the `s1-a` interface for packet
    capture.

 4. Start a "debugging" SMTP server on host `b`:

    ```bash
    b$ sudo python3 -m smtpd -n --class DebuggingServer 0.0.0.0:25
    ```

    This Python SMTP server simply interacts with clients over SMTP and prints
    the messages it receives to standard output.

 5. Send a message.  On host `a`, execute the following to send an email
    message from host `a` to host `b`:

    ```
    a$ swaks --server 10.0.0.2 --to joe@example.com
    ```

 6. Send a message with attachment.  On host `a`, execute the following to send
    an email message with an attachment from host `a` to host `b`:

    ```
    a$ swaks --server 10.0.0.2 --attach byu-y-mtn2.jpg --to joe@example.com
    ```

 7. Follow TCP Streams.  For the emails sent in #5 and #6, open the
    corresponding TCP stream by following the instructions below:

    1. Find one of the frames that is part of the SMTP interaction.
       Right-click on that frame and select "Follow", then "TCP Stream".  Leave
       open the "Follow TCP Stream" window that is created.
    2. Go back to the Wireshark capture window, and clear the display filter
       (should say "tcp.stream eq 1" or the like).  Then go back to #1 for the
       next email.

## Questions

Consider each of the emails sent above when answering the following questions.
Use the TCP stream windows to help you understand and answer the questions.

 1. With SMTP, who initiates the SMTP conversation - client or server?
 2. What command does the client use to introduce itself?
 3. What command does the client use to send the actual email headers and
    message body?
 4. How does the client indicate to the server that it is done sending the
    email message?
 5. What numerical response codes did the server return for the `MAIL FROM`,
    `RCPT TO`, `DATA`, and `QUIT` commands?
 6. Briefly describe the makeup of image attachment in the second email, as
    seen "on the wire".
 7. What must the server do to display the image properly?
