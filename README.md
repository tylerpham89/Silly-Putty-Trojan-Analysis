**Static analysis**

1.  **Hash identification:**

-   Test putty file:

-   ![](images/media/image2.png){width="6.29826990376203in"
    height="3.996978346456693in"}

-   Official putty file:

-   ![](images/media/image1.png){width="5.921875546806649in"
    height="4.104471784776903in"}

-   Running certutil -hashfile \<file path\> MD5/SHA1/SHA256 gives us
    the respective hashes.

-   The hash of the Silly Putty binary does not match the official PuTTY
    binary's hash. This indicates the Silly Putty binary has been
    modified or tampered with compared to the legitimate PuTTY
    executable.

2.  **Architecture analysis:**

-   ![](images/media/image15.png){width="5.510960192475941in"
    height="3.9739588801399823in"}

-   Lets load the silly putty executable into PE-bear (a PE viewer
    provided by flare vm)

-   Click file hdr, and it takes me to this screen for the test putty
    file:

-   ![](images/media/image9.png){width="5.411862423447069in"
    height="3.9114588801399823in"}

-   Official executable:

-   ![](images/media/image10.png){width="5.588542213473316in"
    height="4.003330052493438in"}

-   At the top, we can see the value for the machine is 14c, meaning
    that it is 32 bit (x86) binary. If it was 64 bit, then it would have
    said 8664. We can also read in the characteristics that is is a 32
    bit word machine

-   The official architecture differs in comparison to the official
    executable

3.  **virusTotal analysis:**

-   Test putty file:

-   ![](images/media/image3.png){width="5.601342957130359in"
    height="4.005208880139983in"}

-   ![](images/media/image13.png){width="5.570459317585302in"
    height="4.276042213473316in"}

-   Official putty file:

-   ![](images/media/image8.png){width="5.4915485564304465in"
    height="3.901042213473316in"}

-   ![](images/media/image14.png){width="5.50738188976378in"
    height="3.8347692475940507in"}

-   Running the test file into virusTotal and looking at the summary
    gives us 60/72 security vendors flagging the file as malicious and
    its main threat is a trojan.

-   Comparing this to the official executable we can see that it is
    infected.

4.  **String analysis:**

-   I am going to use strings and convert the strings into a text file
    using strings \<file path.exe\> \> \<filepath.txt\>

-   ![](images/media/image6.png){width="5.953125546806649in"
    height="3.539437882764654in"}

-   We can see below that the test putty file has some suspicious things
    going on like messing with system32, attempting to grab the username
    data, sending passwords with camouflage packets, etc.

-   ![](images/media/image11.png){width="5.442661854768154in"
    height="3.6383552055993in"}

-   

-   ![](images/media/image5.png){width="5.471757436570429in"
    height="3.6741447944007in"}

-   ![](images/media/image4.png){width="5.495624453193351in"
    height="3.7277307524059493in"}

**Dynamic analysis**

1.  **Behavior Analysis (Static Behavior Comparison):**

-   I then open wireshark before I open the test putty file to analyze
    the network traffic. (i changed my network on my vm to host only)

-   ![](images/media/image20.png){width="5.370568678915135in"
    height="4.067708880139983in"}

-   ![](images/media/image16.png){width="5.053226159230096in"
    height="3.7656255468066493in"}

-   Unfortunately, i capture now network traffic going out or anything
    suspicious. It appears I must do something inside of the putty for
    it to trigger.

-   However, when I launched the putty executable, I did notice that
    something interesting happens for a split second.

-   ![](images/media/image7.png){width="5.772587489063867in"
    height="4.505208880139983in"}

-   For a split second, it opens powershell. Im going to examine the
    executable in proc mon

-   ![](images/media/image12.png){width="5.830439632545931in"
    height="4.036458880139983in"}

-   ![](images/media/image18.png){width="5.661458880139983in"
    height="3.873629702537183in"}

-   Inside proc mon, I apply these filters to see what it is doing
    inside of powershell

-   I see multiple powershell processes

-   ![](images/media/image19.png){width="5.309102143482065in"
    height="3.5989588801399823in"}

-   Filtering by the pid of the test putty file, we see the following:

-   ![](images/media/image17.png){width="5.195875984251969in"
    height="3.505551181102362in"}

-   Unfortunately, I did not capture anything out of the ordinary. I
    most likely would have to do something inside of the putty file like
    ssh or something.

-   I dont have a server to ssh in

2.  **Network Traffic Analysis:**

-   Unable to

summary

1.  The Silly Putty binary's hash (MD5/SHA256) is different from the
    official PuTTY binary's known hash. This mismatch indicates the
    Silly Putty file has been modified or tampered with compared to the
    legitimate PuTTY executable.

2.  Using PE-bear, the Silly Putty binary is 32-bit (Machine: 0x14c).
    The official PuTTY in this lab appears to have a different
    architecture, confirming they're not identical builds.

3.  On VirusTotal, the Silly Putty sample was heavily flagged (e.g.,
    \~60/72 antivirus engines) as a Trojan or similar malware. By
    contrast, the official PuTTY binary generally shows no or very few
    detections.

4.  Running strings on the Silly Putty binary revealed unusual
    references to system paths, possible attempts to access user data,
    and mention of powershell-related commands. These strings do not
    appear in the official PuTTY binary, indicating malicious or
    trojanized functionality.

5.  Process Monitor and Process Explorer revealed that Silly Putty can
    spawn multiple PowerShell processes---sometimes
    momentarily---suggesting a hidden payload. Official PuTTY never
    launches PowerShell or any secondary processes in normal usage. This
    behavior indicates the Silly Putty binary is likely trojanized and
    performing unauthorized actions in the background.

6.  In Wireshark captures, no immediate or clearly malicious outbound
    connections were seen (only routine Windows multicast/broadcast
    traffic). However, official PuTTY typically only initiates traffic
    to the SSH server specified by the user. The lack of suspicious
    connections in your capture might mean the malicious component
    requires additional triggers or the payload is primarily local
    (e.g., launching PowerShell) rather than network-based.
