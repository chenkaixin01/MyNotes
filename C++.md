launch.json
```
{

    "version": "0.2.0",

    "configurations": [

        {

            "name": "(Windows) Launch",

            "type": "cppvsdbg",

            "request": "launch",

            "program": "${workspaceFolder}/build/bin/Debug/sd.exe",

            "args": ["-m","D:/models/stable-diffusion/v2-1_768-ema-pruned.q8_0.gguf","-p","\"A pure white sheep\"","-o","D:/output/white.png"],

            "stopAtEntry": false,

            "cwd": "${workspaceFolder}/build/bin/Debug",

            "environment": [],

            "console": "externalTerminal"

        }

    ]

}
```

