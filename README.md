## sol_fa_salad[dot]py

## language: python3

#############################################################
##  CREATE the AUDIO
##    Create the HEADER
##      Open the OUTPUT file
##        WRITE the HEADER
##          WRITE the AUDIO
##            CLOSE the OUTPUT file
#############################################################
# CREATE AUDIO

## First, set things up:

list_pitch = []
flt_x = 13.75
flt_y = 2**(1/12)
int_n = 0

while int_n < 18:
    list_pitch.append(flt_x)
    flt_x = flt_x * flt_y
    int_n += 1

int_tempo = 160
int_newtempo = 160
int_measure=4
int_newmeasure=4
int_beat=4
int_newbeat=4
int_note=3
int_duration=0
int_this_duration=0
int_action=0
int_first_blood = 0
flt_sample_count = 0.0
int_bar_count = 0
int_old_bar_count = 0
int_position = 0
int_samples_per_wave = 0
int_bar_total = 0
flt_tick_samples = 0.0

list_raw = []

int_maxvoice = 9  ## 0 through 9 is a total of ten voices
list_voice = [[0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5],\
              [0,135,0,3276.0,1.0,0.5]]

########## 0  [duration (integer, 44100 = 1 second),
########## 1    period (integer),
########## 2     position (integer),
########## 3      amplitude (float),
########## 4       mix (float, 0.0 through 1.0,
##########               relative amplitude of SQUARE),
########## 5        pan (float, 0.0 through 1.0, 0.0 = left,
##########            0.5 = middle, 1.0 = right)
##########
##########  x 10 voices (0 through 9)

list_newvoice = [[0,135,0,3276.0,1.0,0.5],\
                 [0,135,0,3276.0,1.0,0.5],\
                 [0,135,0,3276.0,1.0,0.5],\
                 [0,135,0,3276.0,1.0,0.5],\
                 [0,135,0,3276.0,1.0,0.5],\
                 [0,135,0,3276.0,1.0,0.5],\
                 [0,135,0,3276.0,1.0,0.5],\
                 [0,135,0,3276.0,1.0,0.5],\
                 [0,135,0,3276.0,1.0,1.0],\
                 [0,135,0,3276.0,1.0,1.0]]

list_change = [[0,0,0,0,0,0],\
               [0,0,0,0,0,0],\
               [0,0,0,0,0,0],\
               [0,0,0,0,0,0],\
               [0,0,0,0,0,0],\
               [0,0,0,0,0,0],\
               [0,0,0,0,0,0],\
               [0,0,0,0,0,0],\
               [0,0,0,0,0,0],\
               [0,0,0,0,0,0]]

############################################################
############################################################

flt_duration = 0.0
flt_new_duration = 0.0

int_action = 0
int_duration = 0

## Now we gotta mess with the text file:

print("Enter path and filename of existing manuscript file.")
filename = input()
print("Enter path and filename of music file to be created.")
musicfilename = input()


print("****************************************************")

prog_file = open(filename, "r")

xline = prog_file.readline()

bad_error = 0    

while xline != "" and bad_error == 0:

    if xline[-1] == chr(10):
        xline = xline[:-1]
        
    print(xline)
    #print("=")

    int_n = xline.find("*") # remove anything after "*"
    if int_n > -1:
        xline = xline[:int_n]

    xline = xline.lower()  ##  change to lowercase
    xline = xline.replace(" ","")  #  Remove spaces
    xline = xline.replace("_","")  #  Remove underscores      

    ## Here we go with a big decoding mess:

## blank line
    if xline == "":
        # Do nothing!
        dummy = 0
        #print("Nothing in this line.")
        
## silence (integer in milliseconds)
    elif xline[0:8] == "silence:":
        xline = xline[8:]

        if xline.isdigit():
            int_ms = float(xline)
        else:
            print(">>> SILENCE parameter DOES NOT COMPUTE! Sorry 'bout that.")
            bad_error = 1

        int_samples = round((44100 * int_ms)/1000)
        while int_samples > 0:
            list_raw.append(0)
            list_raw.append(0)
            list_raw.append(0)
            list_raw.append(0)
            int_samples -= 1
        int_action=0

## silent samples (in number of samples)
    elif xline[0:15] == "silent samples:":
        xline = xline[15:]

        if xline.isdigit():
            int_samples = int(xline)
        else:
            print(">>> SILENT SAMPLES parameter DOES NOT COMPUTE! Sorry 'bout that.")
            bad_error = 1

        while int_samples > 0:
            list_raw.append(0)
            list_raw.append(0)
            list_raw.append(0)
            list_raw.append(0)
            int_samples -= 1
        int_action=0
## tempo
    elif xline[0:6] == "tempo=":
        xline = xline[6:]
        if xline.isdigit():
            int_newtempo = int(xline)
            #print("TEMPO =", int_tempo)  #  <-------------------!!!
        else:
            print(">>> TEMPO parameter DOES NOT COMPUTE! Sorry 'bout that.")
            bad_error = 1
        int_action=0
## measure
    elif xline[0:8] == "measure=":
        xline = xline[8:]
        if xline.isdigit():
            int_newmeasure = int(xline)
            #print("MEASURE =", int_newmeasure)  #  <-------------------!!!
        else:
            print(">>> MEASURE parameter DOES NOT COMPUTE! Sorry 'bout that.")
            bad_error = 1
        int_action=0
## beat
    elif xline[0:5] == "beat=":
        xline = xline[5:]
        if xline.isdigit():
            int_newbeat = int(xline)
            #print("BEAT =", int_newbeat)  #  <-------------------!!!
        else:
            print(">>> BEAT parameter DOES NOT COMPUTE! Sorry 'bout that.")
            bad_error = 1
        int_action=0
## pan
    elif xline[0:3] == "pan":      
        xline = xline[3:]
        int_n = xline.find("=")
        if int_n == -1:
            print(">>> Syntax error using PAN! Sorry 'bout that.")
            bad_error = 1
        else:
            yline = xline[:int_n]
            if yline.isdigit():
                int_x = int(yline)
                if int_x < 0 or int_x > int_maxvoice:
                    print(">>> PAN voice OUT OF RANGE! Sorry 'bout that.")
                    bad_error = 1
                else:
                    xline = xline[int_n + 1:]
                    try:
                        flt_xx = float(xline)
                        #print("PAN:",int_x, ",", flt_xx)  #  <-------------------!!!
                        list_newvoice[int_x][5] = flt_xx
                        list_change[int_x][5] = 1
                    except:
                        print(">>> PAN parameter DOES NOT COMPUTE! Sorry 'bout that.")
                        bad_error = 1
            else:
                print(">>> PAN voice DOES NOT COMPUTE! Sorry 'bout that.")
                bad_error = 1
        int_action=0
## mix
    elif xline[0:3] == "mix":
        xline = xline[3:]
        int_n = xline.find("=")
        if int_n == -1:
            print(">>> Syntax error using MIX! Sorry 'bout that.")
            bad_error = 1
        else:
            yline = xline[:int_n]
            if yline.isdigit():
                int_x = int(yline)
                if int_x < 0 or int_x > int_maxvoice:
                    print(">>> MIX voice OUT OF RANGE! Sorry 'bout that.")
                    bad_error = 1
                else:
                    xline = xline[int_n + 1:]
                    try:
                        flt_xx = float(xline)
                        #print("MIX:",int_x, ",", flt_xx)  #  <-------------------!!!
                        list_newvoice[int_x][4] = flt_xx
                        list_change[int_x][4] = 1
                    except:
                        print(">>> MIX parameter DOES NOT COMPUTE! Sorry 'bout that.")
                        bad_error = 1
            else:
                print(">>> MIX voice DOES NOT COMPUTE! Sorry 'bout that.")
                bad_error = 1
        int_action=0
## bar
    elif xline[0:3] == "bar":   
        flt_tick_samples = float((60 * 44100) / (int_tempo * int_beat))
        int_bar_total = (int_measure * int_beat)

        int_x = int_bar_total - int_bar_count
        ## NOTE: int_bar_count cannot be greater than (int_bar_total - 1)
        ##         nor less than zero

        if int_x < 0:
            bad_error = 1
            print("Something's wrong. Too many beats in the measure? This should not happen!")
        elif int_x > int_bar_total:
            bad_error = 1
            print("Something's wrong but not sure what. Oh, well...")
        else:
            flt_sample_count = float(flt_sample_count + (int_x * flt_tick_samples))
            int_duration = int(flt_sample_count)
            flt_sample_count = float(flt_sample_count - int_duration)
            int_bar_count = 0
            int_old_bar_count = 0
        if int_first_blood == 0:
            int_duration = 0
            int_first_blood = 1
        int_action=2  ##  We're gonna DO something!
        
## integer
    elif xline[0].isdigit() and (int_first_blood == 1):
        int_tempo = int_newtempo      
        zline = xline
        int_bar_total = (int_measure * int_beat)
        int_beatno = 1
        int_tickno = 1
        int_n = zline.find(")")
        if int_n == -1:
            print(">>> Syntax error: Parenthesis needed after number! Sorry 'bout that.")
            bad_error = 1
        else:
            yline = zline[:int_n]
            zline = zline[int_n+1:]
            #print("yline =", yline, "zline =", zline)  ##  <---------------------------------!!!
            int_n = yline.find(":")
            if int_n > -1:
                vline=yline[:int_n]
                wline=yline[int_n+1:]
                #print("vline =", vline, "wline =", wline)  ##  <-----------------------------!!!
                if vline.isdigit():
                    int_beatno = int(vline)
                    if int_beatno > int_measure:
                        print("Too many beats in the measure!")
                        bad_error = 1
                else:
                    print(">>> Number was expected before colon.")
                    bad_error = 1

                if wline.isdigit():
                    int_tickno = int(wline)
                    if int_tickno > int_beat:
                        print("Too many sub-beats in the beat!")
                        bad_error = 1
                else:
                    print(">>> Number was expected before parenthesis.")
                    bad_error = 1
            else:
                if yline.isdigit():
                    int_beatno = int(yline)
                else:
                    print(">>> Number was expected before parenthesis.")
                    bad_error = 1

            #print("BEAtNO =", int_beatno, "SUBBEATNO =", int_tickno, "zline:", zline)  ##  <----------------!!!
            int_bar_count = ((int_beatno - 1) * int_beat) + (int_tickno - 1)
            if int_bar_count < int_old_bar_count:
                bad_error = 1
                print("Hey there--something went wrong because the timing isn't right.")
            if int_bar_count >= int_bar_total:
                bad_error = 1
                print("Wow! Something went wrong because the timing isn't right.")

                
            voice_count = 0
            int_note = 3
            int_octave = 0
            
        while (zline != "") and (bad_error == 0):
            
            int_n = zline.find(",")

            if int_n == -1:
                yline=zline
                zline=""
            elif int_n == 0:
                zline=zline[1:]
                yline=""
            else:
                yline = zline[:int_n]
                zline = zline[int_n + 1:]

            #print("VOICE:", voice_count, "yline:", yline, "zline:", zline)

            if yline != "":
                if not yline[0].isdigit():
                    print(">>> Syntax error: Expected number isn't there! Sorry 'bout that.")
                    bad_error = 1
                else:
                    int_note = 3
                    int_octave = int(yline[0])
                    if int_octave > 5:
                      print(">>> Octave too high in voice",voice_count,"(range is 0 through 5).")
                      bad_error = 1

                #####  >>> sol-fa pitches for pure scales <<<  #####

                    if yline[1:3] == "do":
                        int_note = 180
                        if int_octave > 4:
                            int_note = 90
                    elif yline[1:4] == "&re":
                        int_note = 162
                        if int_octave > 4:
                            int_note = 81
                    elif yline[1:3] == "re":
                        int_note = 160
                        if int_octave > 4:
                            int_note = 80
                    elif yline[1:3] == "mi":
                        int_note = 144
                        if int_octave > 4:
                            int_note = 72
                    elif yline[1:3] == "fa":
                        int_note = 135
                        if int_octave > 4:
                            bad_error=1
                            print("Pitch is too darned high!!! Can't do it, dag-nabbit!")
                    elif yline[1:3] == "fi":
                        int_note = 128
                        if int_octave > 4:
                            bad_error=1
                            print("Pitch is too darned high!!! Can't do it, dag-nabbit!")
                    elif yline[1:3] == "so":
                        int_note = 120
                        if int_octave > 4:
                            bad_error=1
                            print("Pitch is too darned high!!! Can't do it, dag-nabbit!")
                    elif yline[1:3] == "la":
                        int_note = 108
                        if int_octave > 4:
                            bad_error=1
                            print("Pitch is too darned high!!! Can't do it, dag-nabbit!")
                    elif yline[1:3] == "ti":
                        int_note = 96
                        if int_octave > 4:
                            bad_error=1
                            
                #####  >>> pitches for standard tuning  <<<  #####
                            
                    elif yline[1] == "c":
                      int_note = 3
                    elif yline[1] == "d":
                      int_note = 5
                    elif yline[1] == "e":
                      int_note = 7
                    elif yline[1] == "f":
                      int_note = 8
                    elif yline[1] == "g":
                      int_note = 10
                    elif yline[1] == "a":
                      int_note = 12
                    elif yline[1] == "b":
                      int_note = 14
                    else:
                        print(">>> Something in the text doesn't belong here.")
                        bad_error = 1

                if (int_note > 20) and (bad_error == 0):
                    if int_note > 90:
                        int_octave +=1
                    while int_octave < 5:
                        int_note = int_note + int_note
                        int_octave +=1
                    int_samples_per_wave = int_note
                        
                    int_n = yline.find(":")
                    if int_n == -1:
                        print(">>> Syntax error: Expected colon isn't there! Sorry 'bout that.")
                        bad_error = 1                     
                    else:
                        yline = yline[int_n + 1:]
                  
                    if yline.isdigit():
                        int_this_duration = int(yline)
                    else:
                        print(">>> Syntax error: Expected number isn't there! Sorry 'bout that.")
                        bad_error = 1
                        
                elif (int_note <= 20) and (bad_error == 0):
                    if yline[2] == "-":
                         int_note -= 1
                    if yline[3] == "-":
                         int_note -= 1
                    if yline [2] == "+":
                         int_note += 1
                    if yline [3] == "+":
                         int_note += 1
                         
                    int_n = yline.find(":")
                    if int_n == -1:
                        print(">>> Syntax error: Expected colon isn't there! Sorry 'bout that.")
                        bad_error = 1                     
                    else:
                        yline = yline[int_n + 1:]
                  
                    if yline.isdigit():
                        int_this_duration = int(yline)
                    else:
                        print(">>> Syntax error: Expected number isn't there! Sorry 'bout that.")
                        bad_error = 1

                    ###  I guess I have what I need to start stringing stuff together.

                    ###  I've got the expected number of the octave,
                    ###    the number of the pitch,
                    ###      and the duration of the int_note.

                    flt_pitch = list_pitch[int_note]

                    while int_octave > 0 and bad_error == 0:
                         flt_pitch = flt_pitch + flt_pitch
                         int_octave -= 1

                    int_samples_per_wave = round(44100 / flt_pitch)
#########################################################################################################################
                flt_tick_samples = float((60 * 44100) / (int_tempo * int_beat))
                
                int_x = int(((int_this_duration - 0.25) * flt_tick_samples)/ int_samples_per_wave)
                int_adjusted_duration = round(int_samples_per_wave * int_x)

                list_newvoice[voice_count][0] = int_adjusted_duration
                list_change[voice_count][0] = 1
                list_newvoice[voice_count][1] = int_samples_per_wave
                list_change[voice_count][1] = 1
                list_newvoice[voice_count][2] = 0
                list_change[voice_count][2] = 1

                ### There! I stuck them in!
                ############################

            voice_count += 1

        int_this_bar_count = int_bar_count - int_old_bar_count
        ## NOTE: int_bar_count cannot be greater than (int_bar_total - 1)
        ##         nor less than zero

        if int_this_bar_count < 0:
            bad_error = 1
            print("Something's wrong. This should not happen!")
        elif int_this_bar_count >= int_bar_total:
            bad_error = 1
            print("Something's wrong but not sure what. Oh, well...")
        else:
            flt_sample_count = float(flt_sample_count + (int_this_bar_count * flt_tick_samples))
            int_duration = int(flt_sample_count)
            flt_sample_count = float(flt_sample_count - int_duration)

        int_old_bar_count = int_bar_count       

        int_action=1 ##  DO SOMETHING!

    elif xline[0].isdigit() and (int_first_blood == 0):
        bad_error=1
        print('A "BAR" must come first before notes can appear. Sorry about that.')

    else: ## This captures any nonsense GARBAGE text.
        ########
        print(">>> Something somewhere doesn't work.")
        bad_error = 1
        int_action=0

    ### Do your thing here
    ###########################
    ###########################
########## 0   duration (integer, 44100 = 1 second),
########## 1    period (integer),
########## 2     position (integer),
########## 3      amplitude (float),
########## 4       mix (float, 0.0 through 1.0,
##########               relative amplitude of SQUARE),
########## 5        pan (float, 0.0 through 1.0, 0.5 = middle)
##########
##########  x 10 voices (0 through 9)
    ###########################
    ###########################           

    if (bad_error == 0) and (int_action != 0):
        #######
        while (int_duration > 0) and (int_action != 0) and (bad_error ==0):
            #############
            flt_xleft = 0.0
            flt_xright = 0.0
            int_voice = 0

            while int_voice <= int_maxvoice:
                if list_voice[int_voice][0] > 0:
                    int_samples_per_wave = list_voice[int_voice][1]
                    int_position = list_voice[int_voice][2]
                    flt_amplitude = float(list_voice[int_voice][3])
                    flt_sqr = float(list_voice[int_voice][4])
                    flt_rmp = float(1 - flt_sqr)
                    flt_right = list_voice[int_voice][5]
                    flt_left = float(1 - flt_right)
                    int_x = ((int_samples_per_wave - int_position) - int_position) - 1
                    
                    flt_rmpx = float(int_x / (int_samples_per_wave - 1))
                    flt_xleft = flt_xleft + (flt_amplitude * flt_rmpx * flt_rmp * flt_left)
                    flt_xright = flt_xright + (flt_amplitude * flt_rmpx * flt_rmp * flt_right)

                    flt_y = flt_amplitude * flt_sqr
                    if int_x > 0:
                        flt_xleft = flt_xleft + (flt_left * flt_y)
                        flt_xright = flt_xright + (flt_right * flt_y)
                    elif int_x < 0:
                        flt_xleft = flt_xleft - (flt_left * flt_y)
                        flt_xright = flt_xright - (flt_right * flt_y)
                        
                    int_x =list_voice[int_voice][0] - 1
                    list_voice[int_voice][0] = int_x
                    int_position += 1
                    if int_position > (int_samples_per_wave - 1):
                        int_position = 0
                    list_voice[int_voice][2] = int_position
                int_voice += 1            
            #############

            int_left = round(flt_xleft)
            int_a = int_left % 256
            int_m = int_left // 256
            int_b = int_m % 256

            int_right = round(flt_xright)
            int_c = int_right % 256
            int_m = int_right // 256
            int_d = int_m % 256

            list_raw.append(int_a)
            list_raw.append(int_b)
            list_raw.append(int_c)
            list_raw.append(int_d)
            
            int_duration -= 1
        ###########################
    if int_action == 1:
        int_i = 0
        while int_i <= int_maxvoice:
            if list_change[int_i][0] == 1:  ##  remaining duration of note
                list_voice[int_i][0] = list_newvoice[int_i][0]
                list_change[int_i][0] = 0
            if list_change[int_i][1] == 1:  ##  period of pitch
                list_voice[int_i][1] = list_newvoice[int_i][1]
                list_change[int_i][1] = 0
            if list_change[int_i][2] == 1:  ##  position of pointer within waveform
                list_voice[int_i][2] = list_newvoice[int_i][2]
                list_change[int_i][2] = 0
            if list_change[int_i][4] == 1:  ##  mix of square vs ramp
                list_voice[int_i][4] = list_newvoice[int_i][4]
                list_change[int_i][4] = 0
            if list_change[int_i][5] == 1:  ##  pan
                list_voice[int_i][5] = list_newvoice[int_i][5]
                list_change[int_i][5] = 0
            int_i += 1
            
    if int_action == 2:
        int_tempo = int_newtempo
        int_measure = int_newmeasure
        int_beat = int_newbeat
            
    int_action = 0

## Now we're back dealing with the text file
    


    xline = prog_file.readline()
    
prog_file.close()

print("*************************************")
print("** >>>",filename)
if bad_error != 0:
    print("** >>>  Error  <<<")












## All done with audio!                

#########################################
#########################################
#########################################
#########################################

if bad_error==0:

    int_audiolength = len(list_raw)

    #"RIFF"
    list_header = [ord("R"), ord("I"), ord("F"), ord("F")]

    ##  length till END OF FILE
    ##   which is length of AUDIO PORTION + 36 (32 bit integer):

    int_w = 36 + int_audiolength
    list_header.append(int_w % 256)
    int_x=int_w // 256
    list_header.append(int_x % 256)
    int_y=int_x // 256
    list_header.append(int_y % 256)
    int_z=int_y // 256
    list_header.append(int_z % 256)

    #"WAVEfmt "
    list_header.append(ord("W"))
    list_header.append(ord("A"))
    list_header.append(ord("V"))
    list_header.append(ord("E"))
    list_header.append(ord("f"))
    list_header.append(ord("m"))
    list_header.append(ord("t"))
    list_header.append(ord(" "))

    #misc
    list_header.append(16)
    list_header.append(0)
    list_header.append(0)
    list_header.append(0)

    list_header.append(1)
    list_header.append(0)
    list_header.append(2)
    list_header.append(0)

    # Sample Rate (44100):
    # 44100 = 68 + (172 * 256)

    list_header.append(68)
    list_header.append(172)

    #misc
    list_header.append(0)
    list_header.append(0)
    list_header.append(1)
    list_header.append(177)

    list_header.append(2)
    list_header.append(0)
    list_header.append(4)
    list_header.append(0)

    list_header.append(16)
    list_header.append(0)

    #"data"

    list_header.append(ord("d"))
    list_header.append(ord("a"))
    list_header.append(ord("t"))
    list_header.append(ord("a"))

    ##  length of AUDIO portion (32 bit integer):

    int_w = int_audiolength
    list_header.append(int_w % 256)
    int_x=int_w // 256
    list_header.append(int_x % 256)
    int_y=int_x // 256
    list_header.append(int_y % 256)
    int_z=int_y // 256
    list_header.append(int_z % 256)

    ##  END of HEADER

    #########################################
    #########################################
    #########################################
    #########################################

    bin_header_image = bytearray(list_header)

    bin_audio_image = bytearray(list_raw)
    f_output=open(musicfilename ,"wb")
    f_output.write(bin_header_image)
    f_output.write(bin_audio_image)
    f_output.close()

    print()
    print("**",musicfilename, "created!")
    print("** All done! Now listen to the MUSIC!")
