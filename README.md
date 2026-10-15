:: MAKE SURE THIS FILE AND ALL FOLDERS ARE ON THE C DRIVE ITSELF, NOT INSIDE A FOLDER
:: Install windows adk first 
:: Then download the following packages from microsoft and place them in the updates folder
https://catalog.sf.dl.delivery.mp.microsoft.com/filestreamingservice/files/b986f1b3-b24f-4302-85d4-baa913ed3a26/public/windows11.0-kb5064081-x64_a1096145ded3adfc26f8f23442281533429f0e38.msu

https://catalog.sf.dl.delivery.mp.microsoft.com/filestreamingservice/files/fa84cc49-18b2-4c26-b389-90c96e6ae0d2/public/windows11.0-kb5054156-x64_a0c1638cbcf4cf33dbe9a5bef69db374b4786974.msu

https://catalog.sf.dl.delivery.mp.microsoft.com/filestreamingservice/files/518df5fb-59fb-43d7-a44f-990e150b948a/public/windows11.0-kb5124010-x64_95a6272121df38221bfafc90b50baa8ec3e5294e.msu

https://catalog.sf.dl.delivery.mp.microsoft.com/filestreamingservice/files/94520a88-858f-4832-a57d-7211f6d84a4e/public/Windows11.0-KB5121794-x64_5e20a3cce48d6611b16bdec42b07167f5456586a.msu

:: Next, mount a 24h2 IoT LTSC iso and copy all the files to C:\ISO_Work
:: Check which index is IoT LTSC (should be 2 if downloaded from MAS)
dism /Get-WimInfo /WimFile:C:\ISO_Work\sources\install.wim
:: If 2 is not the correct index number make sure to change it
dism /mount-wim /wimfile:C:\ISO_Work\sources\install.wim /index:2 /mountdir:C:\Mount

:: Latest 24H2 cumulative update (meets EP prerequisites)
dism /image:C:\Mount /add-package /packagepath:"C:\Updates\windows11.0-kb5064081-x64_a1096145ded3adfc26f8f23442281533429f0e38.msu"

:: Enablement package 24H2 → 25H2
dism /image:C:\Mount /add-package /packagepath:"C:\Updates\windows11.0-kb5054156-x64_a0c1638cbcf4cf33dbe9a5bef69db374b4786974.msu"

:: Latest 25H2 cumulative update (meets EP prerequisites)
dism /image:C:\Mount /add-package /packagepath:"C:\Updates\windows11.0-kb5124010-x64_95a6272121df38221bfafc90b50baa8ec3e5294e.msu"

:: Enablement package 25H2 → 26H2
dism /image:C:\Mount /add-package /packagepath:"C:\Updates\Windows11.0-KB5121794-x64_5e20a3cce48d6611b16bdec42b07167f5456586a.msu"


:: Load registry hive
reg load HKLM\OFFLINE C:\Mount\Windows\System32\config\SOFTWARE   

:: Disable checking for updates during install (you should unplug your ethernet cord regardless until you run the update disabling tool)
reg add "HKLM\OFFLINE\Policies\Microsoft\Windows\WindowsUpdate" /v DoNotConnectToWindowsUpdateInternetLocations /t REG_DWORD /d 1 /f   

:: Optional enabling of the classic context menu
reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve   

:: Unload registry hive

reg unload HKLM\OFFLINE

:: Old version cleanup
dism /image:C:\Mount /cleanup-image /startcomponentcleanup

dism /unmount-image /mountdir:C:\Mount /commita


:: Delete other versions by exporting only the desired image
:: If 2 is not the correct index number make sure to change it
dism /export-image /SourceImageFile:"C:\ISO_Work\sources\install.wim" /SourceIndex:2 /DestinationImageFile:"C:\ISO_Work\sources\install_new.wim" /Compress:max /CheckIntegrity    
:: !! ONCE EXPORT IS COMPLETE DELETE THE OLD INSTALL.WIM AND RENAME INSTALL_NEW.WIM TO INSTALL.WIM !!

:: Export iso with oscdimg
oscdimg -m -o -u2 -udfver102 -bootdata:2#p0,e,b"C:\ISO_Work\boot\etfsboot.com"#pEF,e,b"C:\ISO_Work\efi\microsoft\boot\efisys.bin" -l"Win11_26H2_IoT_LTSC" "C:\ISO_Work" "C:\Win11_26H2_IoT_LTSC.iso"   
