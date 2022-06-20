from controller import Robot

robot = Robot()

timestep = int(robot.getBasicTimeStep())

dis_sen = []
dis_sen_names = ["ds_left", "ds_right", "ds_mid"]
dis_val = [0]*len(dis_sen_names)
 
for name in dis_sen_names:
    dis_sen.append(robot.getDevice(name))
    dis_sen[-1].enable(timestep)

motors = []
motor_names = ["wheel1", "wheel2", "wheel3", "wheel4"]

for name in motor_names:
    motors.append(robot.getDevice(name))
    motors[-1].setPosition(float('inf'))
    motors[-1].setVelocity(0.0)

error = 0
prop = 0
integ = 0
dif = 0
kp = 0.05
ki = 0
kd = 0.2

def pid(err):
    global error, prop, integ, dif, kp, ki, kd
    prop = err
    integ = err + integ
    dif = err - error
    adjust = (kp*prop) + (ki*integ) + (kd*dif)
    error = err
    return adjust

def set_speed(b_speed, adjust):
    motors[0].setVelocity(b_speed - adjust)
    motors[1].setVelocity(b_speed + adjust)
    motors[2].setVelocity(b_speed - adjust)
    motors[3].setVelocity(b_speed + adjust)

while robot.step(timestep) != -1:
    print("LEFT : {} MID : {} RIGHT : {}".format(dis_sen[0].getValue(),dis_sen[1].getValue(),dis_sen[2].getValue()))
    
    for x in range(len(dis_sen)):
        dis_val[x] = dis_sen[x].getValue()
        if 1000 == dis_val[0] or 1000 == dis_val[1]:
            if dis_val[0] > 500 and dis_val[1] > 500  and dis_val[2]==1000:
                 err = 1000 - dis_val[2]
                 rectify = pid(err)
                 set_speed(5, rectify)
                 
            elif dis_val[0] < 500 and dis_val[1] > 500 and dis_val[2]==1000:
                 err = 1000 - dis_val[0]
                 rectify = pid(err)
                 set_speed(5, -rectify)
                 
            elif dis_val[0] > 500 and dis_val[1] < 500 and dis_val[2]==1000:
                 err = 1000 - dis_val[1]
                 rectify = pid(err)
                 set_speed(5, rectify)
                 
            elif dis_val[0]==1000 and dis_val[1] < 500 and dis_val[2] < 500:
                 err = 1000 - dis_val[1]
                 rectify = pid(err)
                 set_speed(8, -rectify)
                 
            elif dis_val[1]==1000 and dis_val[0] < 500 and dis_val[2] < 500:
                 err = 1000-dis_val[0]
                 rectify = pid(err)
                 set_speed(8, rectify)

        else:
            if dis_val[2] != 1000:
                set_speed(0, 4)
            else:
                err = 1000 - dis_val[2]
                rectify = pid(err)
                set_speed(5, rectify)