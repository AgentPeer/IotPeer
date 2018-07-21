### A Scalable Multiagent Architecture for monitoring IoT devices

> In this project we provide the algorithms used to extends the DHT (using agents) to aid it when scalability issues will arise in its nodes.

### Frontier device responsibilities

**[D1]** Update Frontier Process.
```javascript
di.UPDATE_FRONTIER() {
  myNeighbors = getNeighbors(now); // request neighbor information
  for each device viz in myNeighbors {
    deviceList.add(viz, viz.location);
  }
  sendUpdates(agent, di, deviceList);

  if (ServiceModule.canActAsFrontierDevice(di) == false) {
    for each device viz in myNeighbors
      if (viz.isFrontier == true) {
        accept = sendSubstitution(viz);
        if (accept) {
          notifyRotation(viz);
          stopUpdates();
        }
      }
  }
}
```

**[D2]** Split Process.
```javascript
di.SPLIT_PROCESS() {
  if (iAmBorderDevice == true) {
     dNext = getNextFrontierDevice();
     if (disconnected(di, dNext) {
        path = greedyAlternativePath(di, dNext);
        if (path == null) {
           sendNewPathNotification(agent, di, dNext, path);
        } else {
           sendSeparationNotification(agent, di, dNext);
        }
     }
  }
}
```

**[D3]** Update Neighbors Process.
```javascript
di.UPDATE_NEIGHBORS(List L) {
  for each device dL in L {
    myNeighbors = getNeighbors(now);
    if (dL in myNeighbors) {
      myNeighbors = myNeighbors + dL;
      otherNeighbors = dL.getNeighbors(); // just return its stored neighhbors
      L.add(otherNeighbors);
      if (length(L) > MAX_NEIGHBORS) { // 50 neighbor devices
        return;
      }
    }
  }
}
```

**[D4]** Agentifying Process.
```javascript
di.DEVICE_AGENTIFICATION(List deviceGroup, List sequentialAgents, agent) {
  if (ServiceModule.canActAsAgent(di) == true) {
    iAmAgent = true;
    createsAgent(deviceGroup, sequentialAgents, agent);
    for each device viz in myNeighbors
        accept = sendSubstitution(viz);
        if (accept) {
          notifyRotation(viz);
        }
    }
    stopUpdates(); // stop because agent responsibilities consumes energy
  }
}
```


### Agent responsibilities

