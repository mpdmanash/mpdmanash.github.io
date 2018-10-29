---
title: Plot real-time terminal data
description: A python script to plot 1D realtime data being written to the stdout by other application
header: Plot real-time terminal data
tags: python, matplotlib, stdout, pipe, visualize
comments: true
excerpt: A python script to plot 1D realtime data being written to the stdout by other application
---

> A python script to plot 1D real-time data being written to the stdout by other application

When we develop software to process data, there is often a need to quickly visualize it for debug purposes. One most common option is to write the data to the stdout of the terminal. For example:
- `print data` in Python
- `std::cout << data << std::endl;` in C++

<br>
However, sometimes plotting the data makes more sense. In this article, I'll show an example of how to develop a simple tool to plot such data in real-time.

----

### Python script to plot multiple sets of 1D data
The script should be able to 1) read data from the terminal and 2) plot the data. One way to read from the terminal is to use [**pipe**](https://www.linfo.org/pipes.html) to redirect the data from the application writing the data to this script. 
<br> For example: `$ python some_application.py | python plot_script.py`

Once the data is being redirected, we can read it in python by the following script (plot_script.py):
{% highlight python linenos %}
import sys

def main():
    while True:
        try:
            data = sys.stdin.readline()
            sys.stdout.write(data)
            sys.stdout.flush()
        except KeyboardInterrupt:
            print('exiting')
            sys.exit()
main()
{% endhighlight %}
You may try running it as: `$ ping 127.0.0.1 | python plot_script.py`
<br>  
'data' variable in the above script has to be plotted now. In this article, I'll show you how to parse multiple streams of 1D data. We first generate such synthetic data for the sake of demonstration (synthetic-data.py):

{% highlight python linenos %}
import sys
import math
import time
from random import randint

i = 0
while True:
    x = math.sin(float(i)*3.14/180)
    y = math.cos(float(i)*3.14/180)
    z = -x
    print str(x)+","+str(y)+","+str(z)
    sys.stdout.flush()
    i = i+1
    time.sleep(0.01)
{% endhighlight %}
with comma separated output. Here I demonstrate 3 sets of 1D data (x,y,z) also represented as variables.
<pre>
> 0.0,1.0,-0.0
> 0.0174435597087,0.999847849537,-0.0174435597087
> 0.0348818113261,0.999391444449,-0.0348818113261
> 0.052309448376,0.99863092362,-0.052309448376
> 0.0697211676123,0.997566518477,-0.0697211676123
> 0.0871116706329,0.99619855292,-0.0871116706329
> 0.104475665491,0.994527443221,-0.104475665491
> 0.121807868308,0.992553697902,-0.121807868308
.
.
</pre>
<br>
Thus, we can parse the 'data' in 'plot_script.py' using `data.split(',')`. After adding the **matplotlib** functions, here is our final 'plot_script.py' script.

{% highlight python linenos %}
"""
# A python script to plot sets of 1D time-series data.
# Multiple sets can be plotted simultaneously.
# Ideally developed to plot setpoint vs system output in Robotic
   P.I.D controllers.

# Usage: python plot.py -t < X axis timesteps > -n < number of variables>
# Tutorial: https://manashpratim.com/plot-realtime-terminal-data

# Author: Manash Pratim Das (mpdmanash@iitkgp.ac.in)
"""

import sys, getopt
from collections import deque
import matplotlib.pyplot as plt


class AnalogPlot:
    # constr
    def __init__(self, maxLen, variables):
        self.datan = []
        for i in range(variables):
            datai = deque([0.0] * maxLen)
            self.datan.append(datai)
        self.maxLen = maxLen
        self.variables = variables

    # add to buffer
    def addToBuf(self, buf, val):
        if len(buf) < self.maxLen:
            buf.append(val)
        else:
            buf.pop()
            buf.appendleft(val)

    # add data
    def add(self, data):
        assert(len(data) == self.variables)
        for i in range(self.variables):
            self.addToBuf(self.datan[i], data[i])

    # update plot
    def update(self, frameNum, an, values):
        try:
            data = values
            self.add(data)
            for i in range(self.variables):
                an[i].set_data(range(self.maxLen), self.datan[i])
        except KeyboardInterrupt:
            print('exiting')
        return

    # clean up
    def close(self):
        pass


def main(argv):
    # plot parameters
    timesteps = 1000
    variables = 1

    try:
        opts, args = getopt.getopt(argv,"ht:n:")
    except getopt.GetoptError:
        print 'python plot.py -t < X axis timesteps > -n < number of variables>'
        sys.exit()
    for opt, arg in opts:
        if opt == '-h':
            print 'python plot.py -t < X axis timesteps > -n < number of variables>'
            sys.exit()
        elif opt == '-t':
            timesteps = int(arg)
        elif opt == '-n':
            variables = int(arg)

    print "Using timesteps =", timesteps 
    print "Number of variables =", variables
    analogPlot = AnalogPlot(timesteps, variables)
    fig = plt.figure()
    ax = plt.axes(xlim=(0, timesteps))
    an = []
    for i in range(variables):
        ai, = ax.plot([], [])
        an.append(ai)

    x = 1
    fig.show()

    while True:
        try:
            data = sys.stdin.readline()
            sys.stdout.write(data)
            sys.stdout.flush()
            parts = data.split(',')
            if len(parts) >= variables:
                values = []
                for i in range(variables):
                    vi = float(parts[i])
                    values.append(vi)
                analogPlot.update(x, an, values)
                x = x + 1
                ax.relim()
                ax.autoscale_view(True,False,True)
                plt.draw()
            else:
                print "Invalid number of variables"
        except KeyboardInterrupt:
            print('exiting')
            sys.exit()

    analogPlot.close()
    print('exiting.')

main(sys.argv[1:])
{% endhighlight %}

This script accepts two arguments:
- `-t`: The length of X axis. Since the input data is a time-series data, each input is considered a timestep. Thus this variable determines the number of past timesteps to plot from the current time.
- `-n`: Number of variables to plot in the input data. The respective values for the different variables in a timestep are presented in a comma-separated string. Thus, this script can plot multiple 1D data variables.

To visualize the synthetic data we generated before.   
Run `$ python synthetic-data.py | python plot_script.py -t 1000 -n 3`

<br>
Here is the graph output with different color for different variables:
![example-plot](/img/example-plot.png "example-plot")