## go build linux binary on windows

cmd:
```
set GOARCH=amd64
set GOOS=linux
go build
```

powershell:
```
$env:GOOS="linux"; go build
```

background excute on linux:
```
./binary &
```

!!! Reference

    [https://stackoverflow.com/questions/20829155/how-to-cross-compile-from-windows-to-linux](https://stackoverflow.com/questions/20829155/how-to-cross-compile-from-windows-to-linux)