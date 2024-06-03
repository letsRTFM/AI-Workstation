Dual 3090 AI Inference Workstation

## Videos

- TBD

## Hardware

- Power Supply: [Corsair 1000W SFF-L](https://amzn.to/459azXN) *Afilliate Link  
  __Warning__ I'm getting a bit of coil whine under load with this PSU and graphics card combination
- Motherboard: [BD790i](https://store.minisforum.com/products/minisforum-bd770i)
- CPU: [CPU AMD Ryzen 9 7945HX 5.4 GHz - 16 Cores 32 Threads](https://browser.geekbench.com/v6/cpu/4875003) *Built into motherboard
- RAM: [Crucial 5600 96GB Kit](https://amzn.to/49RVpaD) *Affiliate Link
- Case: [Geometric Future Model 8](https://amzn.to/457XwWw) *Affiliate Link
- Storage: [Crucial T705 4TB Gen 5 NVME](https://amzn.to/459Zh5p) *Affiliate Link
- GPU: 2x [Nvidia 3090 Founders Edition](https://cdn.mos.cms.futurecdn.net/RtAnnCQxaVJNYgA4LbBhuJ-970-80.png) *Purchased Used on Ebay
- Network Adapters:
  - 2.5Gbps Realtek NIC (built into motherboard) 
  - (Optional) 10Gbps [AQC107 NIC in nvme form factor](https://www.aliexpress.us/item/3256804879089176.html)  
    *No Driver needed on rocky linux 9, autodetects as Aquantia Ethernet
- Cooling: 
  - 5 x [Noctua NF-A12x25](https://amzn.to/3KnKQRU) *Affiliate Link
  - 3 x [Noctua NF-A14](https://amzn.to/3X1i2Gl) *Affiliate Link
  - 1 x [Noctua NF-A12x15](https://amzn.to/3uNb0Ju) *Affiliate Link
- Screws:
  - M2.5 [Screws for Minisforum CPU Fan Bracket](https://amzn.to/4djLXPE) *Affiliate Link
  - (Optional) [Misc Assorted PC Screws](https://amzn.to/4b4fPxQ) *Affiliate Link
- Cables
  - 1 x [JMT PCI-E 4.0 x16 1 to 2 PCIe Bifurcation](https://amzn.to/3VmQtWC) *Affiliate Link
  - 1 x [PCIE 4.0 Extension Cable Length 250mm](https://amzn.to/4aJrO3a) *Affiliate Link
  - 1 x [EZDIY-FAB Vertical GPU Mount with High-Speed PCIE 4.0 Riser Cable](https://amzn.to/3X5NjI0) *Affiliate Link
  - 2 x [6pin + 2pin PCIe Power Extension Cables](https://amzn.to/4aJrIbO) *Affiliate Link
  - 1 x [USB 3.1 Type-E to Type C USB3.0 Motherboard Header Adapter Male to Female](https://www.aliexpress.us/item/3256806775660644.html)
  - 1 x [USB C Extension Cable](https://amzn.to/3X6QLCh) *Affiliate Link
  - 1 x [USB Connector USB Extension Cable USB2.0 to 9Pin Conector 9 Pin Male to External USB A](https://www.aliexpress.us/item/3256804933848801.html)
 

## Software

- AI Text Generation: [llama.cpp](https://github.com/ggerganov/llama.cpp)
- Editor: [VSCode](https://code.visualstudio.com/)
- Editor Extensions: 
  - LLM Integration [Continue](https://github.com/continuedev/continue)

## Configuration

### Install any Nvidia Drivers and Cuda Dependencies:

https://docs.nvidia.com/cuda/cuda-installation-guide-linux/  
Reboot

### Disable Xorg binding to nvidia cards:

You will find that xorg places processes on your GPUs rather than using the iGPU on our CPU.
This can cause out of memory errors when running AI workloads.

To prevent this you can comment out the x-org configuration found within
/etc/X11/xorg.conf.d/ This will cause x-org not to see the nvidia driver and therefore it won't use it for window management
```
#Section "OutputClass"
#    Identifier "nvidia"
#    MatchDriver "nvidia-drm"
#    Driver "nvidia"
#    Option "AllowEmptyInitialConfiguration"
#    Option "PrimaryGPU" "no"
#    Option "SLI" "Auto"
#    Option "BaseMosaic" "on"
#EndSection

Section "OutputClass"
    Identifier "intel"
    MatchDriver "i915"
    Driver "modesetting"
EndSection
```

### Install Llama.cpp
clone llama.cpp

`git clone https://github.com/ggerganov/llama.cpp.git`  
Reference: https://github.com/ggerganov/llama.cpp  
Compile llama.cpp with nvidia support  

`make LLAMA_CUDA=1`

### Load the models
Download Mixtral GGUF quant  
https://huggingface.co/TheBloke/Mixtral-8x7B-v0.1-GGUF/resolve/main/mixtral-8x7b-v0.1.Q5_K_M.gguf?download=true  
Reference: https://huggingface.co/TheBloke/Mixtral-8x7B-v0.1-GGUF/tree/main  

Download Dolphin Starcoder2 quant:  
https://huggingface.co/bartowski/dolphincoder-starcoder2-15b-GGUF/resolve/main/dolphincoder-starcoder2-15b-Q4_K_M.gguf?download=true  
reference: https://huggingface.co/bartowski/dolphincoder-starcoder2-15b-GGUF  

Place the models into the llama cpp models folder

### Start the llama.cpp server instances

Start the Instruct Server

`./server -m models/mixtral-8x7b-instruct-v0.1.Q5_K_M.gguf -ngl 33`

CUDA VRAM Usage: 31GB

Start the Autocomplete Server

`./server --port 8081 -m models/dolphincoder-starcoder2-15b-Q4_K_M.gguf -ngl 41`

CUDA VRAM Usage:10GB


TOTAL VRAM Usage: 41GB / 48GB

### Configure VScode

Install VScode  
[VSCode](https://code.visualstudio.com/)

Install VScode Extension "Continue"  
https://github.com/continuedev/continue

Configure continue  
Open the configure configuration using the vscode command Palette  
Reference: https://docs.continue.dev/reference/Model%20Providers/llamacpp

``` json
{
  "models": [
    {
      "title": "Mixtral 8x7B",
      "provider": "llama.cpp",
      "model": "mistral-8x7b",
      "apiBase": "http://localhost:8080",
      "systemMessage": "You are an expert software developer. You give helpful and concise responses. if asked to write something like a function, comment or docblock wrap it in code ticks for easy copy paste"
    }
  ],
  "customCommands": [
    {
      "name": "test",
      "prompt": "{{{ input }}}\n\nWrite a comprehensive set of unit tests for the selected code. It should setup, run tests that check for correctness including important edge cases, and teardown. Ensure that the tests are complete and sophisticated. Give the tests just as chat output, don't edit any file.",
      "description": "Write unit tests for highlighted code"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Dolphin Starcoder2",
    "provider": "llama.cpp",
    "model": "starcoder2:15b",
    "apiBase": "http://localhost:8081",
    "useCopyBuffer": false,
    "maxPromptTokens": 4000,
    "prefixPercentage": 0.5,
    "multilineCompletions": "always",
    "debounceDelay": 150
  },
  "allowAnonymousTelemetry": false
}
```
