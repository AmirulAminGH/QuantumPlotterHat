import asyncio
import re
import os
import time
from time import sleep
import websockets
import websocketRTV
import time
import re
import serial
from datetime import datetime
from threading import Thread
import random
import threading
import math
def gettime():
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    return current_time
connected = set()
progress=10
consoleMessage="CONSOLE"
consoleAppend=""
xposmin=90.0
xposmax=10.0
yposmin=7.0
yposmax=90.0
incrementx=(xposmin-xposmax)/80
incrementy=(yposmax-yposmin)/152
xpos=90.0
ypos=7.0
gcodeArray=""
serialInbound=""
ThreadClearance=1
currentspeed="F10000"
Mposx=""
Mposy=""
currentx=0.00
currenty=0.00
OTP=0
GlobalPercentage=0

Angle10=0
Angle20=0
Angle30=0
Angle40=1
Angle50=1
Angle60=1
Angle70=1
Angle80=1
Angle90=1
AngleLimit=30
#update section start
prev_input = None  # initialize previous input to None


def is_different_from_previous_input(current_input):
    global prev_input  # access global variable

    if current_input != prev_input:
        prev_input = current_input  # update previous input
        return True

    return False
#update section end

port = serial.Serial(port='COM9',baudrate=115200,)
#port.open

globalcounter =0
def SerialConnect(command):
    global globalcounter
    global port
    global serialInbound
    serialInbound=""
    #print(command)
    port.write(str.encode(command))  # Send g-code block
    sleep(0.1)
    while port.inWaiting()>0:
        serialInbound+=port.readline().decode('utf-8')
        #serialInbound+=str(globalcounter)
        #globalcounter+=1
        #print(inbound.decode('utf-8'))
        #print(inbound)
    return serialInbound

def functionClassifier(message):
    global xpos
    global ypos
    global incrementy
    global incrementx
    if message =="CLEARLOG":
        print("Log Cleared")
        return("log cleared")
    if message =="UPDATE":
        print("DAQ updated")
        return("log updated")


def extract_substring(string, start, end):
    start_index = string.find(start) + len(start)
    end_index = string.find(end, start_index)
    return string[start_index:end_index]

def PlotBegin():
    global port
    global GlobalPercentage
    global Mposx
    global Mposy
    print("plot thread started")
    global gcodeArray
    global ThreadClearance
    global currentspeed
    localspeed = currentspeed
    print(SerialConnect(localspeed+"\n"))
    while True:
        websocketRTV.breakloop=0
        while websocketRTV.threadStatus==0:
            sleep(0.1)
            pass
        localArray = gcodeArray
        websocketRTV.breakloop =0
        print("plotting process started")
        print(localArray)
        print(SerialConnect("$X\n"))
        print(SerialConnect(localspeed + "\n"))
        error = 0
        ArrayLength=len(localArray)
        loopCount=0
        for idg in range(ArrayLength):

            if websocketRTV.breakloop==1:
                break
            if localspeed !=currentspeed:
                localspeed=currentspeed
                SerialConnect(localspeed+"\n")
            GlobalPercentage = int((idg / ArrayLength) * 100)
            print("percentage is {}".format(GlobalPercentage))

            cmd=localArray[idg]
            print("Sending {}".format(cmd))
            print("current line is {}".format(idg))
            start_time = time.time()
            port.write(str.encode(cmd + '\n'))
            grbl_out = port.readline()  # Wait for grbl response with carriage return
            grbl_out=grbl_out.decode('utf-8')
            end_time = time.time()
            response_time = end_time - start_time
            print(response_time)
            if response_time>1 or loopCount>10 :
                port.write(str.encode('?\n'))
                state = port.readline()
                state=state.decode('utf-8')
                print(state)
                if "MPos:" in state:
                    mposition = re.findall(r'-?\d+(?:\.\d+)?', state)
                    Mposx = mposition[0]
                    Mposy = mposition[1]
                loopCount=0

            print(' : ' + str(grbl_out))

            if "error" in str(grbl_out):
                error+=1
            if "ok" in str(grbl_out):
                loopCount+= 1
            print ("loop count is {}".format(loopCount))
            print("total error {}".format(error))

            while websocketRTV.threadStatus == 0:
                sleep(0.1)
                pass

            #print(SerialConnect(gcodeArray[idg]))
        print("finish sending")
        while True:
            sleep(1)
            postStatus = SerialConnect("?\n")
            if "Bf:" in postStatus:
                postOrder = int(extract_substring(postStatus, "Bf:", ","))
                print("current buffer is {}".format(postOrder))
                print(postStatus)
                if postOrder == 35:
                    break
                # input("Press Enter to continue...")
        print("motion is completed")
        GlobalPercentage=100
        websocketRTV.threadStatus=0
        #await websocket.send("PERCENTAGE" + str(100))
        # sleep(5)
        # print(SerialConnect("?\n"))
async def foo(websocket):
    global Mposx
    global Mposy
    global GlobalPercentage
    while True:
        await websocket.send("PERCENTAGE" + str(GlobalPercentage))
        await asyncio.sleep(0.1)
        if websocketRTV.threadStatus == 0:
            state = SerialConnect("?\n")
            #print(state)
            if "MPos:" in state:
                mposition = re.findall(r'-?\d+(?:\.\d+)?', state)
                Mposx = mposition[0]
                Mposy = mposition[1]
                #print("Mposx>>{}".format(Mposx))
                #print("Mposy>>{}".format(Mposy))
            sleep(0.25)
        await websocket.send("MPOSITION" + Mposx+"/"+Mposy+"/")
        await asyncio.sleep(0.1)


def calculate_angle_change(x1, y1, x2, y2, x3, y3):
    # calculate angles of the two lines
    dx1 = x2 - x1
    dy1 = y2 - y1
    angle1 = math.atan2(dy1, dx1)

    dx2 = x3 - x2
    dy2 = y3 - y2
    angle2 = math.atan2(dy2, dx2)

    # calculate angle change between the two lines
    angle_change = angle2 - angle1
    angle_change_degrees = math.degrees(angle_change)

    return angle_change_degrees
def CalculateAngle(gcodeList):
    global Angle10
    global Angle20
    global Angle30
    global Angle40
    global Angle50
    global Angle60
    global Angle70
    global Angle80
    global Angle90
    gcodeMatrix=gcodeList
    for idx in range (len(gcodeMatrix)-2):
        if "G01" in gcodeMatrix[idx] and "G01" in gcodeMatrix[idx+1] and "G01" in gcodeMatrix[idx+2]:
            p1=gcodeMatrix[idx]
            p2=gcodeMatrix[idx+1]
            p3=gcodeMatrix[idx + 2]
            coordp1 = re.findall(r'-?\d+(?:\.\d+)?',p1)
            coordp2 = re.findall(r'-?\d+(?:\.\d+)?', p2)
            coordp3 = re.findall(r'-?\d+(?:\.\d+)?', p3)
            p1x =float( coordp1[1] )
            p1y =float( coordp1[2] )
            p2x =float( coordp2[1] )
            p2y =float( coordp2[2] )
            p3x =float( coordp3[1] )
            p3y =float( coordp3[2] )
            angleChange= abs(calculate_angle_change(p1x,p1y,p2x,p2y,p3x,p3y))
            print(gcodeMatrix[idx+1])
            print("angle>>{}".format(angleChange))
            #for idg in range (8):
            if angleChange == 0 or angleChange ==0.0:
                gcodeMatrix[idx + 1] = gcodeMatrix[idx + 1] + " NULLCHANGE"  # format((idg*10)+10)
            if angleChange >0 and angleChange < 360:
                gcodeMatrix[idx+1]=gcodeMatrix[idx+1] + " ANGELRUSH{}".format(angleChange) #format((idg*10)+10)

    print("ANGULAR MATRIX>>>")
    print(gcodeMatrix)
    print("<<<ANGULAR MATRIX")









async def server(websocket):
    global firstState
    global ThreadClearance
    global consoleMessage
    global progress
    global consoleAppend
    global xpos
    global ypos
    global gcodeArray
    global GlobalPercentage
    global currentx
    global currenty
    global currentspeed
    global Angle10
    global Angle20
    global Angle30
    global Angle40
    global Angle50
    global Angle60
    global Angle70
    global Angle80
    global Angle90

    bulloffsetx=0.00
    bulloffsety=0.00
    BullId=0
    asyncio.create_task(foo(websocket))
    gcodeArray=""
    # await websocket.send("&&&" " &&&"+str(progress)+"%")
    async for message in websocket:
        #print(SerialConnect("$\n"))
        print("message is {}".format(message))
        if "command" in message:
            message=message.replace("command","")
            if message=="CLEARLOG":
                consoleMessage="CONSOLE"
            consoleText = functionClassifier(message)
            consoleMessage =consoleMessage + " >> " + consoleText + " [" + gettime() + "]" + "<br>"
            #await websocket.send("&&&" + consoleMessage + "&&&" + str(progress) + "% &&&" + str(xpos) + "% &&&" + str(ypos) + "% &&&")
            await websocket.send(consoleMessage)
            progress = progress + 1
            print("msg >>" + message)
        if "filestart" in message:
            GlobalPercentage=0
            print(SerialConnect("?\n"))
            print("G-code received")
            message = message.replace("filestart", "")
            message = message.replace("fileend", "")
            filename=re.search('#&%(.*)%&#', message)
            filename=filename.group(1)
            print(filename)
            message = message.replace(filename,"")
            message = message.replace("#&%","")
            message = message.replace("%&#","")
            with open("gcodefiles/{}".format(filename), "w") as f:
                f.write(message)
            with open("gcodecmd2.txt", "w") as f:
                f.write(message)
            tempArray=message.split("\r\n")
            layerarray=""
            CommandList=[]
            xnow="0"
            ynow="0"
            for idx in range(len(tempArray)):
                 stateCheck=""
                 if "G01" in tempArray[idx]:
                    coords = re.findall(r'-?\d+(?:\.\d+)?', tempArray[idx])
                    # xval=extract_substring(tempArray[idx],"X","Y")
                    # yval=extract_substring(tempArray[idx],"Y","\d")
                    xval=coords[1]
                    yval=coords[2]
                    layerarray+=xnow+" "+ynow+" "+xval+" "+yval+" "+"1 1,"
                    xnow=xval
                    ynow=yval
                    stateCheck="G01"
                 if "G00" in tempArray[idx]:
                    coords = re.findall(r'-?\d+(?:\.\d+)?', tempArray[idx])
                    # xval = extract_substring(tempArray[idx],"X","\d")
                    # yval = extract_substring(tempArray[idx],"Y"," ")
                    xval = coords[1]
                    yval = coords[2]
                    layerarray += xnow + " " + ynow + " " + xval + " " + yval + " " + "2 2,"
                    xnow = xval
                    ynow = yval
                    stateCheck="G00"
                 if is_different_from_previous_input(stateCheck)==True:
                     if stateCheck=="G01":
                         CommandList.append("M8")
                         CommandList.append("G4 P1")
                     if stateCheck=="G00":
                         CommandList.append("M9")
                         CommandList.append("G4 P1")
                 CommandList.append(tempArray[idx])



                 # if "G00" in tempArray[idx]:
                 #     xval = float(extract_substring(tempArray[idx], "X", " "))
                 #     yval = extract_substring(tempArray[idx], "Y", " ")
                 #     layerarray += xval + " " + yval + ","

            #print(tempArray)
            #gcodeArray=tempArray
            gcodeArray=CommandList
            for idv in range (len(gcodeArray)):
                gcodeArray[idv]=gcodeArray[idv].strip()
            gcodeArray = [line for line in gcodeArray if line.strip() != ""]
            print("GCODE ARRAY>>>>>>>>>>>>>>>>>>>")
            print(gcodeArray)
            print("<<<<<<<<<<<<<<<<<<<GCODE ARRAY")
            #AngleChange=CalculateAngle(gcodeArray)
            #print("command list is \n")
            #print(CommandList)
            #print(message)
            #print(layerarray)
            await websocket.send("LAYERVIEW" + layerarray)

            PosMatrix=layerarray.split(",")
            #print("POSMATRIX>>")
            #print(PosMatrix)
            wMat = 3
            hMat = len(PosMatrix)
            BullMatrix = [[0 for x in range(wMat)] for y in range(hMat)]
            for idq in range(len(PosMatrix)-1):
                LineData=PosMatrix[idq].split(" ")
                #print(LineData)
                BullMatrix[idq][0]=LineData[0]
                BullMatrix[idq][1] = LineData[1]


        if "FILELIST" in message:
            print("file list updated")
            folder = 'C:/Users/VICTUS/PycharmProjects/Plotter control/gcodefiles'
            # Get the list of files in the folder
            files = os.listdir(folder)
            # Print the list of files
            print(files)
            filelist = "FILELIST File List : "
            for idx in range(len(files)):
                filelist += "<br>" + str(idx + 1) + ". " + files[idx]
            await websocket.send(filelist)
        if "HISTORY" in message:
            with open("history.txt", "r") as f:
                historyheader="HISTORY"
                historydata=f.read()
                await websocket.send(historyheader+historydata)
        if "CAMERA" in message:
            print("camera on")
        if "STARTPLOT" in message:
            websocketRTV.threadStatus=1
        if "STOPPLOT" in message:
            print("plotting process stopped")
            #websocketRTV.threadStatus = 0
            websocketRTV.breakloop=1
            sleep(1)
            #print(SerialConnect("!"))
            # position=SerialConnect("?\n")
            # currentx=extract_substring(position,"MPos:",",")
            # currenty=extract_substring(position,",",",0.000")
            # sleep(1)
            # print(SerialConnect("ctrl-x\n"))
            # sleep(1)
            # poscmd="G92 X"+currentx+" Y"+currenty+"\n"
        if "PAUSEPLOT" in message:
            print("plotting process paused")
            websocketRTV.threadStatus=0
            print(SerialConnect("!"))
        if "RESUMEPLOT" in message:
            print("plotting process resumed")
            websocketRTV.threadStatus=1
            SerialConnect("~")
        if "speed" in message:
            lowspeed="F10000"
            mediumspeed="F30000"
            highspeed="F50000"
            if message=="speedLOW":
                currentspeed=lowspeed
            if message=="speedMEDIUM":
                currentspeed=mediumspeed
            if message=="speedHIGH":
                currentspeed=highspeed
            print("current speed is set to {}".format(currentspeed))
            print(SerialConnect(currentspeed+"\n"))
        if "GCODE" in message:
            if "GCODEOFFSETX" in message:
                offsetx = message.replace("GCODEOFFSETX", "")
                bulloffsetx = float(offsetx)
                gcodeecho2="X offset set to {}".format(offsetx)
            if "GCODEOFFSETY" in message:
                offsety = message.replace("GCODEOFFSETY", "")
                bulloffsety = float(offsety)
                gcodeecho2 = "Y offset set to {}".format(offsety)

            if "GCODEANGLE10" in message:
                newm = message.replace("GCODEANGLE10", "")
                Angle10 = float(newm)
            if "GCODEANGLE20" in message:
                newm=message.replace("GCODEANGLE20","")
                Angle20=float(newm)
            if "GCODEANGLE30" in message:
                newm=message.replace("GCODEANGLE30","")
                Angle30=float(newm)
            if "GCODEANGLE40" in message:
                newm=message.replace("GCODEANGLE40","")
                Angle40=float(newm)
            if "GCODEANGLE50" in message:
                newm=message.replace("GCODEANGLE50","")
                Angle50=float(newm)
            if "GCODEANGLE60" in message:
                newm=message.replace("GCODEANGLE60","")
                Angle60=float(newm)
            if "GCODEANGLE70" in message:
                newm=message.replace("GCODEANGLE70","")
                Angle70=float(newm)
            if "GCODEANGLE80" in message:
                newm=message.replace("GCODEANGLE80","")
                Angle80=float(newm)
            if "GCODEANGLE90" in message:
                newm=message.replace("GCODEANGLE90","")
                Angle90=float(newm)

            gcodemessage = message.replace("GCODE", "")
            gcodemessage+="\n"
            print("sent>>{}".format(gcodemessage))
            gcodeecho=SerialConnect(gcodemessage)
            print("echoed>>{}".format(gcodeecho))
            if "<" in gcodeecho:
                gcodeecho = gcodeecho.replace("<", "")
                gcodeecho = gcodeecho.replace(">", "")
            if "\n" in gcodeecho:
                gcodeecho = gcodeecho.replace("\n","<br>")
            print()
            consoleMessage = consoleMessage + " >> " + gcodeecho + " [" + gettime() + "]" + "<br>"
            #consoleMessage = consoleMessage + " >> " + gcodeecho2 + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "JOGRIGHT" in message:
            jogdistance=extract_substring(message,",",";")
            jogspeed=extract_substring(message,";","~")
            jogcmd=SerialConnect("$J=G91 Y"+jogdistance+"F"+jogspeed+"\n")
            consoleMessage = consoleMessage + " >> " +jogcmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "JOGLEFT" in message:
            jogdistance=extract_substring(message,",",";")
            jogspeed=extract_substring(message,";","~")
            jogcmd=SerialConnect("$J=G91 Y-"+jogdistance+"F"+jogspeed+"\n")
            consoleMessage = consoleMessage + " >> " +jogcmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "JOGFORWARD" in message:
            jogdistance = extract_substring(message, ",", ";")
            jogspeed = extract_substring(message, ";", "~")
            jogcmd = SerialConnect("$J=G91 X" + jogdistance + "F" + jogspeed + "\n")
            consoleMessage = consoleMessage + " >> " + jogcmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "JOGREVERSE" in message:
            jogdistance = extract_substring(message, ",", ";")
            jogspeed = extract_substring(message, ";", "~")
            jogcmd = SerialConnect("$J=G91 X-" + jogdistance + "F" + jogspeed + "\n")
            consoleMessage = consoleMessage + " >> " + jogcmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "PENUP" in message:
            toolcmd=SerialConnect("M9\n")
            consoleMessage = consoleMessage + " >> " + toolcmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "PENDOWN" in message:
            toolcmd = SerialConnect("M8\n")
            consoleMessage = consoleMessage + " >> " + toolcmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "GOHOME" in message:
            homecmd = SerialConnect("$H\n")
            consoleMessage = consoleMessage + " >> " + homecmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "UNLOCK" in message:
            unlockcmd = SerialConnect("$X\n")
            consoleMessage = consoleMessage + " >> " + unlockcmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "SQUARE" in message:
            squarecmd = SerialConnect("M3\n F1000\n G01 X-100\n M5\n")
            consoleMessage = consoleMessage + " >> " + squarecmd + " [" + gettime() + "]" + "<br>"
            await websocket.send(consoleMessage)
        if "GOZERO" in message:
            print(SerialConnect("G00 X0 Y0\n"))
        if "SETZERO" in message:
            print(SerialConnect("G92 X0 Y0 Z0\n"))
        if "SOFTRESET" in message:
            reset="\n" #(ctrl-x)


        if "BULLSEYEnextline" in message:
            BullId=BullId+1
            bullposx=BullMatrix[BullId][0]
            bullposy=BullMatrix[BullId][1]

            await websocket.send("BULLSEYEPOSITION" + bullposx + "/" + bullposy + "/")

        if "BULLSEYEprevline" in message:
            BullId = BullId - 1
            if BullId < 0:
                BullId=0
            bullposx = BullMatrix[BullId][0]
            bullposy = BullMatrix[BullId][1]
            await websocket.send("BULLSEYEPOSITION" + bullposx + "/" + bullposy + "/")
            print("prevline")
        if "BULLSEYEgo" in message:
            gox = BullMatrix[BullId][0]
            goy = BullMatrix[BullId][1]
            SerialConnect("G00 X"+str(gox)+" Y"+str(goy)+" \n")
            print("goline")
        if "BULLSEYEenable" in message:
            enaMsg="G00 X"+str(bulloffsetx)+" Y"+str(bulloffsety)+" \n"
            SerialConnect(enaMsg)
            SerialConnect("G92 X0 Y0 \n")

        if "BULLSEYEdisable" in message:
            SerialConnect("G28 \n")
            SerialConnect("G92 X0 Y0 \n")


            # time.sleep(3)
        # await websocket.send(f'Got a new MSG FOR YOU: {"updating new message hehehe"}')

plotter = threading.Thread(target=PlotBegin)
plotter.start()
print("main thread started")
start_server = websockets.serve(server, "localhost", 5000,max_size= None)
asyncio.get_event_loop().run_until_complete(start_server)
asyncio.get_event_loop().run_forever()



