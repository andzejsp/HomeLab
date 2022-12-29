SOURCE FROM: https://gitlab.com/-/snippets/1950292

# Set up the Linux Host
Install `freerdp-x11` on Ubuntu:
```bash
#!/bin/bash

# Add "universe" repository (needed to install the package "freerdp2-x11")
sudo add-apt-repository universe

# Install xfreerdp (FreeRDP)
sudo apt install freerdp2-x11
```

# Set up the Windows Host
## Configure RemoteFX
As described [here](https://gist.github.com/Misairu-G/616f7b2756c488148b7309addc940b28#remotefx-configure-and-fine-tuning):
1. run `gpedit.msc` through `Win+R`
2. go to `Computer Configuration -> Administrative Templates -> 
Windows Components -> Remote Desktop Service -> Remote Desktop Session Host -> 
Remote Session Environment`
    - enable `Use advanced RemoteFX graphics for RemoteApp`
    - (Optional) enable `Configure image quality for RemoteFX adaptive Graphics`
  and set it to `High`
    - enable `Enable RemoteFX encoding for RemoteFX clients designed for Windows Server 2008 R2 SP1`
    - enable `Configure compression for RemoteFX data`, set it to `Do not use an RDP compression algorithm` (connection compression will result in extra 
  latency, which we're trying to avoid)
3. go to `Computer Configuration -> Administrative Templates -> 
Windows Components -> Remote Desktop Service -> Remote Desktop Session Host -> 
Remote Session Environment -> RemoteFX for Windows Server 2008 R2`
    - enable `Configure RemoteFX`
    - (Optional) enable `Optimize visual experience when using RemoteFX` and set 
  both option to `Highest`.


# Run the test
Connect to your Windows 10 Host:
```bash
#!/bin/bash

# Settings
user="user"
host="192.168.1.10"
size="100%"

# Connect to RDP Server
xfreerdp /u:$user /v:$host /size:$size /bpp:32 +clipboard +fonts /gdi:hw /rfx \
    /rfx-mode:video +menu-anims +window-drag
```

# Enable 60 FPS in RDP (not tested)
If what you've read till now doesn't work you can try the beneath instructions:
1. run `regedit` through `Win+R`
2. go to the path: 
`HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations`
3. create a DWORD Value named `DWMFRAMEINTERVAL`
4. modify it and set:
    - Base: `Decimal`
    - Value Data: `15`
    
# Conclusion
I've also tried the same method on an Ultrabook running Windows 10 (MSI PS42 
8M), which has an iGPU (Intel HD 620):
- w/o RDP (on the Ultrabook): Youtube FHD videos at 60 fps ran smoothly
- w/ RDP (on the Ultrabook): Youtube FHD videos at 60 fps were lagging quite a 
bit, but HD videos at 30 fps were playing just fine.

N.B. For an RDP session on an Ultrabook running a minimal Linux distro with
Openbox (for programming, browsing), I think it would run good enough.

In conclusion, it doesn't depend much on Ethernet speed.

However, a dedicated GPU is recommended to have a better experience.
