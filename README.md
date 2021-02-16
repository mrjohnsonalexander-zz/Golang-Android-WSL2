# Golang-Android-WSL2
Golang native app deployed to Android device, and developed on Windows Subsystem for Linux. 

# Windows Setup
My local is a Windows Surface Pro 7, and followed guidance from https://docs/microsoft.com/windows/wsl-install-win10

## Enable Windows Features
```cmd
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
...
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
...
```

## Set default WSL version 2 for OS
Restarting machine enabled me to download/install Ubuntu 20.04 LTS from Microsoft Store.
I then Downloaded WSL update from https://wslstorage.blob.core.windows.net/wslblob/wsl_update_x64.msi, and set default WSL version.
```cmd
wsl --set-default-version 2
wsl --list
...
wsl --set-version Ubuntu-20.04 2
...
wsl -l -v
```
I then logged into my Ubuntu instance, and verified the kernal version is WSL2.
```bash
uname -a
```
## Start Android Device Emulator
Installed Android Studio from https://developer.android.com/studio, created Nexus_5X_API_29 device, and started emulator.
```cmd
C:\Users\<user>\AppData\Local\Android\Sdk\emulator\emulator.exe -avd Nexus_5X_API_29
```

## Start ADB Server
```cmd
adb kill-server
adb -a nodaemon server start
```
I then verified my emulated device was attached.
```cmd
adb devices
...
```

# WSL Setup
I executed script for installing Android SDK found https://gist.github.com/jjvillavicencio/18feb09f0e93e017a861678bc638dcb0, replacing "\<user\>" with appropriate name, from by Ubuntu bash terminal.
```bash
wget https://gist.githubusercontent.com/jjvillavicencio/18feb09f0e93e017a861678bc638dcb0/raw/0f6cb9c2b7bbd3c2b89e11f22cb29e9ce2b9c810/setup.sh
...
chmod +x setup.sh
```
I updated the directory structure of Android SDK installion, to avoid to sdkmanager warnings described https://stackoverflow.com/questions/60440509/android-command-line-tools-sdkmanager-always-shows-warning-could-not-create-se.
```bash
mkdir ~/Android/cmdline-tools
mv ~/Android/tools ~/Android/cmdline-tools/tools
```

## Install Go, gomobile, Android SDK, Android NDK, ADB, and Socat
```bash
sudo apt install golang
...
go get golang.org/x/mobile/cmd/gomobile
sudo apt install andriod-tools-adb
...
sudo apt install socat
...
wget https://dl.google.com/android/repository/android-ndk-r20-linux-x86_64.zip
...
unzip android-ndk-r20-linux-x86_64.zip
...
```
Fixing or preventing "adb server version doesn't match this client error" can be done copying sdk adb to bin\adb.  
```bash
sudo cp ~/Android/platform-tools/adb /usr/bin/adb
```
## Update and Append .bashrc
The setup.sh script will add ANDRIOD_HOME, so updated it, to match where I moved it to avoid sdkmanager warnings.
```bash
vim .bashrc
export ANDROID_HOME=/home/<user>/Android/cmdline-tools/latest
```
Also appended to .bashrc so the end of my it looks like (replace "\<user\>")
```bash
...
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin

export ANDROID_HOME=/home/<user>/Android/cmdline-tools/latest
export ANDROID_SDK_ROOT=/home/<user>/Android
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/platform-tools

export GOPATH=/home/<user>/go
export PATH=$PATH:$GOPATH/bin

export ANDROID_NDK_HOME=/home/<user>/android-ndk-r20
export PATH=$PATH:$NDROID_NDK_HOME

export WSL_HOST=$(tail -1 /etc/resolv.conf | cut -d' ' -f2)
export ADB_SERVER_SOCKET=tcp:$WSL_HOST:5037
```
Of course needed to source it
```bash
source ~/.bashrc
```

## Build and Install Example Gomobile app
Following guidance here https://pkg.go.dev/golang.org/x/mobile@v0.0.0-20210208171126-f462b3930c8f/example/network.
```bash
go get golang.org/x/mobile/cmd/gomobile
gomobile init
go get -v -d golang.org/x/mobile/example/network
...
gomobile build golang.org/x/mobile/example/network
gomobile install golang.org/x/mobile/example/network
```

The on my Emulator, I nativgated to "Apps & notifications", and "See all <int> apps";  scrolled down to find "network" app, clicked to "App info", and clicked Open to launch a Green screen application (indicating golang.org is reachable when the app first starts, or red otherwise).
