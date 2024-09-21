## Step-by-Step Installation of Go on Flatcar Container Linux

### Download and Install Go

1. **Download Go Binary**
    ```bash
    sudo curl -L -o /tmp/go1.16.15.linux-amd64.tar.gz https://dl.google.com/go/go1.16.15.linux-amd64.tar.gz
    ```
    This command downloads the Go binary tarball to the `/tmp` directory.

2. **Create Installation Directory**
    ```bash
    sudo mkdir -p /opt/go
    ```
    This command creates the directory `/opt/go` where Go will be installed.

3. **Extract Go Binary**
    ```bash
    sudo tar -C /opt/go -xzf /tmp/go1.16.15.linux-amd64.tar.gz --strip-components=1
    ```
    This command extracts the Go binary into the `/opt/go` directory.

4. **Add Go to PATH**
    ```bash
    export PATH=$PATH:/opt/go/bin
    ```

5. **Persist the PATH Change**
    ```bash
    echo 'export PATH=$PATH:/opt/go/bin' >> ~/.bashrc
    ```
    This command attempts to append the Go path to the user's `.bashrc` file. Note that if the file system is read-only, this step might fail.

6. **Reload the Profile**
    ```bash
    source ~/.bashrc
    ```
    This command reloads the profile settings.

7. **Verify Installation**
    ```bash
    go version
    ```
    This command verifies the Go installation by displaying the installed Go version.

### Verification Output
```bash
core@worker1 ~ $ go version
go version go1.16.15 linux/amd64
```
