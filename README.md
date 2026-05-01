# win


```
notepad C:\Windows\System32\drivers\etc\hosts
```


```
$taskName = "dns_sync"
$url = "https://raw.githubusercontent.com/robocode-pb/win/refs/heads/main/dns.txt"

$fullCode = {
    param($url)
    try {
        # Налаштування безпеки та завантаження
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
        $webClient = New-Object System.Net.WebClient
        $webClient.Encoding = [System.Text.Encoding]::UTF8
        
        # Отримуємо дані, розбиваємо на рядки, чистимо від сміття та дублікатів
        $data = $webClient.DownloadString($url)
        $sites = $data -split "`n" | ForEach-Object { $_.Trim() } | Where-Object { $_ -ne "" } | Select-Object -Unique
        
        $hostsPath = "$env:windir\System32\drivers\etc\hosts"
        
        # Читаємо поточний файл hosts
        $currentHosts = Get-Content $hostsPath
        $newLines = @()

        foreach ($site in $sites) {
            $entry = "127.0.0.1 $site"
            # Перевіряємо, чи немає вже такого запису (регістронезалежно)
            if (-not ($currentHosts -match [regex]::Escape($entry))) {
                $newLines += $entry
            }
        }

        # Якщо є що додавати — додаємо з нового рядка
        if ($newLines.Count -gt 0) {
            $textToAdd = "`r`n# Auto-blocked sites`r`n" + ($newLines -join "`r`n")
            Add-Content -Path $hostsPath -Value $textToAdd -Force
        }
    } catch {
        # Помилки ігноруються, щоб не заважати користувачу
    }
}.ToString()

# Кодування в Base64 для стабільної роботи Scheduled Task
$bytes = [System.Text.Encoding]::Unicode.GetBytes($fullCode)
$encoded = [Convert]::ToBase64String($bytes)

# Налаштування завдання
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -WindowStyle Hidden -EncodedCommand $encoded -url '$url'"
$trigger = New-ScheduledTaskTrigger -AtLogOn
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest

# Реєстрація (Force перезаписує старе завдання з таким же іменем)
Register-ScheduledTask -TaskName $taskName -Action $action -Trigger $trigger -Principal $principal -Force

Write-Host "✅ Завдання '$taskName' створено успішно!" -ForegroundColor Green
Write-Host "Скрипт буде автоматично оновлювати hosts при кожному вході в систему." -ForegroundColor Cyan
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
