# Implementing Count-Min Sketch in p4 

This code is originally from https://github.com/nsg-ethz/p4-learning and I give full credit to the copyright holder. 

I have made minor changes to the code to make it run directly on the bmv2 switch, without comfortable tools such as p4run, p4app or mininet. 

Pre-Installation of dependencies are needed (PI, Behavioral Model (BMv2), P4C). 
They can be downloaded at https://github.com/jafingerhut/p4-guide


## Introduction

The objective of this repository is to implement a Count-min Sketch to run on a bmv2 p4 switch. 
To implement the count-min sketch `registers` and `hash` functions are used.
To count packets belonging to a flow, every packet's 5-tuple (src,dst, sport, dport, proto) is hashed. Each hash output indexes a different register and increase its value by 1
In order to read how many entries were counted for a given flow, read the value at every register and evaluate the minimum.

1.  Clone the repository to local 

    ```
    git clone https://github.com/Seoyul-OH/CountMinSketch.git
    ```

2. ```
    cd CountMinSketch
   ```

3. Define veths in your path 

```
sudo ./veth_setup.sh 
```

4. Compile the Count-min Sketch p4 program 

```
p4c-bm2-ss --p4v 16 cm-sketch.p4 -o cm-sketch.json
```

5. Run the switch in background 
```
sudo simple_switch -i 0@veth0 -i 1@veth2 --log-console --thrift-port 9090 cm-sketch.json
```

6. Open a different terminal in cd CountMinSketch, and send packets 
```
sudo python3 send.py --n-pkt 100000 --n-hh 10 --n-sfw 990 --p-hh 0.95
```

(A total of 100000 packets with 10 heavy hitter flows, 990 small flows, and the percentage of heavy hitter packets to all packets is 0.95)


#### Dimensioning

   1. The estimated count value will be the same as the real or bigger.
   2. The error will be not bigger than `real_count` + ϵ*η with probability 1 - δ.  Where ϵ is the error factor, η is the number of packets hashed and δ is the error probability.

 * `Needed number of registers:` ceil(ln(1/δ))
 * `Needed register length:` ceil(e/ϵ)

For example, if we want a ϵ=0.1 and δ=0.05. Registers = 3, length=28.


7. Set hashes of the controller by running the `set_hashes` option 
```
sudo python3 cm-sketch-controller.py --option "set_hashes"
```

8. Run the controller with the `decode` option
```
sudo python3 cm-sketch-controller.py --eps 0.1 --n 100000 --mod 28 --option decode
