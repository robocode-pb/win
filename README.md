# win


```
notepad C:\Windows\System32\drivers\etc\hosts
```


```

$taskName = "dns"
$url = "https://raw.githubusercontent.com/robocode-pb/win/refs/heads/main/dns.txt"

$fullCode = {
    param($url)
    try {
        # Примусовий TLS 1.2 для з'єднання з GitHub
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
        
        $webClient = New-Object System.Net.WebClient
        $webClient.Encoding = [System.Text.Encoding]::UTF8
        
        $data = $webClient.DownloadString($url)
        $sites = $data -split "`n" | ForEach-Object { $_.Trim() } | Where-Object { $_ -ne "" }
        
        $hostsPath = "$env:windir\System32\drivers\etc\hosts"
        $current = Get-Content $hostsPath
        
        $toAdd = @()
        foreach ($site in $sites) {
            $entry = "127.0.0.1 $site"
            if ($current -notcontains $entry) {
                $toAdd += $entry
            }
        }

        if ($toAdd.Count -gt 0) {
            $textToAdd = "`r`n" + ($toAdd -join "`r`n")
            Add-Content -Path $hostsPath -Value $textToAdd -Force
        }
    } catch {}
}.ToString()

# Кодуємо в Base64, щоб уникнути конфліктів символів
$bytes = [System.Text.Encoding]::Unicode.GetBytes($fullCode)
$encoded = [Convert]::ToBase64String($bytes)

# Створення дії та оновлення завдання
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -WindowStyle Hidden -EncodedCommand $encoded -url '$url'"
$trigger = New-ScheduledTaskTrigger -AtLogOn
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest

Register-ScheduledTask -TaskName $taskName -Action $action -Trigger $trigger -Principal $principal -Force

Write-Host "Готово! Завдання 'dns' тепер повністю автономне та використовує TLS 1.2." -ForegroundColor Green
```
