# Memory Analysis - Ransomware | Medium

The Account Executive called the SOC earlier and sounds very frustrated and angry. He stated he can’t access any files on his computer and keeps receiving a pop-up stating that his files have been encrypted. You disconnected the computer from the network and extracted the memory dump of his machine and started analyzing it with Volatility. Continue your investigation to uncover how the ransomware works and how to stop it!

### Solve

Unzipping the provided archive file produces two files:

- `BTLO.txt`: disclaimer not to share the challenge content
- `infected.vmen`: the memory dump of the compromised host

Used the guide [volatility cheatsheet](https://blog.onfvp.com/post/volatility-cheatsheet/), for this.

Running Volatility with the provided `infected.vmem` file, looking for processes (process scan):

```console
$ python3 vol.py -f infected.vmem windows.psscan
```

![[Pasted image 20230605003700.png]]

From the above, we identify two strange processes, namely `@WanaDecryptor` with process ID (PID) `2688` and `3968`, respectively, and parent process ID (PPID) `2732`.

Looking for this parent process ID, we see that the binary `or4qtckT[.]exe` started these processes:

```console
$ python3 vol.py -f infected.vmem windows.psscan | grep '^2732'
```

![[Pasted image 20230605004206.png]]

Drilling down on this process ID reveals an additional `taskdl[.]exe` is the process used to delete files:

```console
$ python3 vol.py -f infected.vmem windows.psscan | grep '2732'
```

![[Pasted image 20230605004340.png]]

Conducting a scan for the command-line, we find the malicious file was first executed from `C:\Users\hacker\Desktop\or4qtckT.exe`:

```console
$ python3 vol.py -f infected.vmem windows.cmdline | grep or4qtckT.exe
```

![[Pasted image 20230605004652.png]]

This is evidently `WannaCry` ransomware.

Finally, looking into the process handles for process ID `2732`, we find the encryption public key `00000000.eky`:

```console
$ python3 vol.py -f infected.vmem windows.handles --pid 2732 | grep File
```

![[Pasted image 20230605005347.png]]