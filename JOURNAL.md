---
title: "Spark Carrier"
author: "emb3rcia"
description: "Carrier board with case for FriendlyElec's CM3588 Plus CPU Board"
created_at: "2024-07-21"
---

# July 21: The big start
Hello again! I know i said in application form my next project would be AR glasses but i got a different idea! What if i made a motherboard? But heres the problem, any desktop-like motherboards are veeeery hard to design and especially build! Then theres costs of parts which will definetely exceed 350$, VRM needing to handle 300+W of power (which if i do wrong can be dangerous both to parts AND to me and other devices on by electrical network!). So i decided i will use ARM processor. First i checked RPi5 processor, buuut they dont sell it to detalical user :<. So i checked processors with similiar power to it and found Rockchip RK3588! Problem? Yup, they dont sell them too, at least not the bare CPU. Only in bulk and only to companies (And i dont need 1000 CPUs! And i dont have a company!). So i found the FriendlyElec's CM3588 Plus, it is the next best thing i can get. It is basically NAS kit - you get the CPU board with ram already on it and get a NAS carrier board, buuuut you dont need to pay and get carrier board! And they are super user friendly, having their's schematics and pinout layout on theirs wiki! So i thought: "What if i madde carrier board instead? I dont need to pay for BGA soldering and i dont need to pay for 1000 processors at the time! So right now, lets stop talking and get to work:


First up: plan. What i want to do and how?

Well, i want it to use the CM3588 Plus, have max configuration of avaliable ports, and have a case that looks like miniature PC case with 2 fans - one intake and one exhaust.

So lets make a prototype BOM:
- FriendlyElec CM3588 Plus (16 GB LPDDR5/64 GB eMMC) 
- 2x USB-C 3.1 gen 1 OTG + DP - but, because it is OTG combo it can work in HOST mode or DEVICE mode - the computer needs to know when to use when, so i asked ChatGPT (as i didn't know the answer) what should i do to get every possible feature out of these ports, he said i need USB3 + DP mux (to know when to use which signal), VBUS switch (to power things only when the computer is the host) and CC Logic (to know who is host and who is device), so add example of models from ChatGPT to the list
 1. 2x HD3SS460
 2. 2x FUSB302
 3. 2x TPS25942
 4. 2x Amphenol FCI 10137064-00021LF
 5. 2x PESD5V0S1UL,315
 6. 2x BLM18AG601SN1D
 7. 2x RC0603JR‑0722RL
 8. 2x GRM21BR71H104KA01L
 9. 2x C2012X7R1A106K125AC
 10. 2x GRM188R72A104KA35D
- 1x USB-A 3.1 gen 1 HOST - good news! it can go directly to port! But i can add an ESD Safety, so add it to the list:
 1. TPD4EUSB30
 2. Port Wurth Elektronik 692122030100
 3. RC0603JR‑0722RL
 4. GRM21BR71H104KA01L
 5. 1x GRM188R72A104KA35D
 6. 1x C2012X7R1A106K125AC
- 2x USB-A 2.0 HOST - The same goes for these ports!
 1. 2x TPD4EUSB30
 2. 2x Port GCT USB1055-GF-L-A
 3. 2x RC0603JR‑0722RL
 4. GRM21BR71H104KA01L
 5. 2x GRM188R72A104KA35D
 6. 2x C2012X7R1A106K125AC
- MicroSD port - it supports up to SDR104 mode, which is great! (i think), i just need port, but i can add ESD for protection
 1. USBLC6
 2. Hirose Connector DM3AT-SF-PEJM5(41)
 3. RC0603JR‑0722RL
 4. RC0603JR‑0710KL
 5. RC0402JR‑0750RL
 6. 4x RMCF0402FT47K0
 7. 1x GRM188R72A104KA35D
 8. 1x C2012X7R1A106K125AC
- 2x HDMI 2.1 output portsA- one can display up to 8K60 and second one up to 4K60 - i dont need anything other than HDMI Type A ports, but i can add protection!
 1. 2x HDMIULC6
 2. 2x TE Connectivity 2485396-1
 3. 2x BLM18AG601SN1D
 4. 2x PESD5V0S1BA
 5. 8x PESD5V0S1UL
 6. 2x RC0603JR‑0722RL
 7. 6x GRM21BR71H104KA01L
 8. 2x GRM155R71H512KA01D
 9. 2x RG3216P-5101-D-T5 (5.1k)
 10. 2x C2012X7R1A106K125AC
 11. 2x RC0402JR‑0750RL
- 1x HDMI input up to 4Kp60 - i can add just HDMI type A port, but i can add protection!
 1. HDMIULC6
 2. TE Connectivity 2485396-1
 3. BLM18AG601SN1D
 4. PESD5V0S1BA
 5. 4x PESD5V0S1UL
 6. RC0402JR‑0750RL
 7. 3x GRM21BR71H104KA01L
 8. 2x GRM155R71H512KA01D
 9. RG3216P-5101-D-T5 (5.1k)
 10. C2012X7R1A106K125AC
- I wont add MIPI interfaces, cause i dont have anything to test them with, but i will add the audio ports it supports! We have the stereo headphone output (20mW/CH, THD+N <= -80dB, 160Ohm Load) and single-end microphone input, i will add them seperately instead of combo-jack! So we need to add DC blocking capacitor for headset audio, and we can add - you guessed it - protection! Also we need AC coupling capacitor for microphone audio
 1. EEU-FR1E101B
 2. GRM188R71C105KA12D
 3. USBLC6-2SC6
 4. ACJM-3501-SM-2 (mic)
 5. SJ1-3533N (stereo)
 6. 2x RC0603JR‑0722RL
 7. 2x BLM18AG601SN1D
 8. 1x GRM188R72A104KA35D
- Now behold: THE PCIE LANES! there are couple avaliable configuration that pcie can run in, but i want to use the one that gives 1x PCIe Gen 3 x4 and 2x PCIe Gen 2.1 x1, the gen 3 one will be as M.2 drive slot, the PCIe Gen 2.1 x1 will be avaliable as slots, it will be probably the most difficult thing to do, due to needing precise trace lenghts and controlled impendations
 1. TE Connectivity 2199119-3
 2. 2x Molex 87715-9002
 3. 3x PESD5V0S1UL
 4. 3x MF-MSMF050-2
 5. 3x BLM18AG601SN1D
 6. 5x GRM21BR71H104KA01L
 7. TUSB1002RGER
 9. 3x C2012X7R1A106K125AC
 10. 3x GRM188R72A104KA35D
- And last but not least: Ethernet RJ45 2.5 Gb/s port! CM3588 Plus was built for NAS purposes with their NAS carier board, so we can use the on-board Realtek RTL8125B
 1. Pulse Electronics JT4-2504CHL
 2. 2x PESD4ETH1G-T
 3. RC0402JR‑070RL
 4. GRM21BR71C105KA01L
 5. GRM188R72A104KA35D
- Also - UART debugging
 1. FTSH‑105‑01‑L‑DV‑K

I am like 3.5 hours into making this BOM and i am still researching everything i need to add between ports and pins


okay! i made it, full prototype BOM! it has theoretically all the parts that i need (i fricking hope so). i will make the schematics etc tomorrow. rn i am going to rest from this abomination XD

## Time spent this session: 4 hours