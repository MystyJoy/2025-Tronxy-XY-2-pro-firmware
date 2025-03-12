I have the tronxy xy-2 pro, although these instructions should work for any of the tronxy machines.
These instructions have been made with the hope of preventing the weeks of tears, toil and trouble that I have been through. Hope it helps.

My Tronxy XY-2 pro would shift in the y-layer as I would print. I tested and checked every mechanical aspect that would cause such an issue, but to no avail. So I assumed that the tronxy stock firmware some how got corrupted (apperently this can 'just happen').

For some reason, tronxy are very cagey about their firmware (despite it all being open source and mostly stolen from marlin!), so the only files they offer are their newest bootloader and firmware that do not work! It's firmware tronxy! Not nuclear launch codes!
I installed the bootloader and firmware tronxy was offering without backing up the old bootloader and firmware. DO NOT make that mistake, as although the old firmware did not work, it is still incredibly nerve wracking not having it to fall back onto. I do not recommend removing your bootloader at all, there is no need for. I was folllowing instructions that should never have been made.

I've tried many different firmwares that can be found online for the tronxy, most I struggled to get to compile and those that would compile would not upload to the printer.

The files which I used, and managed to get to work are all from [tronxy3d](https://github.com/tronxy3d/F103_PIC480x320)


## How to compile

  1. Install [vscode](https://code.visualstudio.com/), and install platformio plugin in the vscode extension. Do this by clicking on the sie menu option called 'extensions'. Then search for platformIO IDE, install it. (should have a large logo with an ants head)
  2. Download this firmware and unzip it, you will get a firmware folder. Run vscode -> file -> Open Folder, select the firmware folder, open it.
  
 
  3. Expand the folder called Marlin (should look like this:  >Marlin) and then open the file TronxyMachine.h
  4.  Find #define TRONXY_ PROJ (around line 23), change the following project name to PROJ_ XXX (XXX represents your printer model)
    - Note: The modified project name must be a name defined above the file(TronxyMachine.h). If there is no such name, contact customer service for handling
    e.g. your model is XY2_PRO:
<img align="center" width=473 src="buildroot/share/pixmaps/tronxy/modify_model.png" />

  5. Launch platformio by clicking on the ant head in the left-hand column.
  6.  Attempt to compile the firmware by clicking on Default/Build All. The first compilation may take a long time.
  7.  WHEN the firmware fails to compile, if the error message in the terminal is as follows:
<br/>     os.makedirs(state_dir)
<br/>File "<frozen os>", line 215, in makedirs
<br/>File "<frozen os>", line 225, in makedirs
<br/>FileNotFoundError: [WinError 3] The system cannot find the path specified: 'E:\\'
![Drive_location_error](https://github.com/user-attachments/assets/a3718862-4170-412f-a3c6-ef7ef5240b20)
<br/> Open the platformio.ini file and for (around line 15):
core_dir    =E:/.platformio
Change 'E' to 'C', or whatever letter is used for your drive.
![Drive_file_location](https://github.com/user-attachments/assets/fa4939c5-21d1-45a9-af43-b9de6ce36f24)
![drive_fix](https://github.com/user-attachments/assets/e7199e59-691c-471a-a136-fe24df26efdf)

  9. With that changed, attempt to compile again, this time it will take longer than last as packages install themselves. Don't worry if a majority of them fail, wait for the program to finish.
  10. If you get an error message that looks like the following:
      ![Loop_Issue](https://github.com/user-attachments/assets/c2926d7c-f611-45bb-9092-7c6b2f6c420d)
With your cursor, hover over the last link given in the terminal. It will be on the last yellow line that starts with 'file:', hold ctrl and click on it. This will open marlin.py (or you can locate the file manually)
![loop_location](https://github.com/user-attachments/assets/53f95253-96ac-44d8-ba87-0663a3d6f98b)
Locate the highlighted section. User [eoyilmaz](https://community.platformio.org/t/marlin-runtimeerror-deque-mutated-during-iteration/34661/4) solved this issue by replaceing the loop sequence highlighted above to: <br/>

    def replace_define(field, value):
	    found_define = None
	    for define in env['CPPDEFINES']:
		    if define[0] == field:
			    found_define = define
			    break
	    if found_define:
		    env['CPPDEFINES'].remove(found_define)
	    env['CPPDEFINES'].append((field, value))
<br/>You can comment out the old function, and add the new code beneath it. It should look like:
![loop_fixed](https://github.com/user-attachments/assets/6574c2b3-5634-4a6e-8a61-f98e89b119ef)

  11. Compile again and await the next error message.<br/>
  The next message should be:
![Handler_error](https://github.com/user-attachments/assets/0751ac40-b510-469e-84e0-cb89b0c81a4d)
This is because the IRQHandler has been defined in two different places. Hold ctrl and click the link which will open the interrupt.cpp file. <br/> Go to the line indicated e.g. line 349 for EXTI2, it will be different if the error message calls a different function (e.g. for EXTI1 on line 339) so make sure you make the change for the correct block of code.
It will most likely be EXTI2 that is defined in multiple places, but do check.
![IQR-location](https://github.com/user-attachments/assets/54b990e3-dc36-4640-bd80-f97adf803c24)


Comment out the block of code using //


![IQR_CODE_FIXED](https://github.com/user-attachments/assets/51cd289b-0df8-4316-a446-1b64e99c4e09)

 12. Compile again and you shall recieve an error message saying 
![elf-message](https://github.com/user-attachments/assets/cfd4eae6-ce40-4443-8c53-77d25ed4b753)


<br/> the solution to this can be found [here](https://github.com/tronxy3d/F4xx-SIM480x320/issues/17). Ignore the comments saying not to use this solution, as it does work and it does not disable your ability to use the sd card slot to update firmware multiple times, as I have uploaded firmware multiple times using the sd card slot since making this change.
![IQR_fix](https://github.com/user-attachments/assets/d2bb6963-573c-4541-855d-fc88cc8db293)

<br/>As directed, open ini\stm32f4.ini and comment out (using #) or delete line 659 like this:
![IQR_Code_Fix](https://github.com/user-attachments/assets/0a772723-6eac-41ca-b076-852ebff091ed)

<br/>Next do the same thing for ini\stm32f1.ini, commenting out line 476 like this:
![IQR-2-code-fix](https://github.com/user-attachments/assets/5e820697-b2e0-4bc4-9d08-377a7c7882bc)

  13. Once again, compile the firmware. <br/>
     <br/>
  14.The firmware should now be successfully compiled, if not - go to end of document
![success](https://github.com/user-attachments/assets/4e235b87-171e-4025-9229-7db3f3b6bc2e)

## Uploading firmware

  1. As stated before:
![elf_solution](https://github.com/user-attachments/assets/b58fb9db-5e3f-48a0-a6f9-f116cf8a5b0d)
<br/> open the firmware file on your computer, click on '.pio', then 'build', then 'tronxy_stm32f446', then copy the file labelled 'firmware.bin' and paste it into the folder called 'update'
  2. Inside the update folder, delete the file named fmw_tronxy.bin, and then rename firmware.bin to fmw_tronxy.bin (I do not know how necessary this step is, but I did it anyways)
  3.    Copy the 'update' folder into the root directory of the SD card, insert the card into the printer, restart, and the machine will automatically update the firmware. After that, the machine will run the current firmware.<br/> 

## Firmware still not compiling?
As a last attempt, open the firmware folder on your computer, and delete the folder named, '.pio' <br/> Then try to compile the firmware again in VScode. The .pio folder is automatically generated when the code compiles. <br/> If successful, go onto the "Uploading firmware" section of this document. <br/> 
<br/> If you are unsuccessful, goodluck.


