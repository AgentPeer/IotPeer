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
  sendUpdates(agent, di, deviceList); // call agent.UPDATE_PROCESS(deviceList)

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
           sendSeparationNotification(agent, di, dNext); // call agent.SPLIT_PROCESS(di, dNext)
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
di.DEVICE_AGENTIFICATION(List devicesSwarm, List sequentialAgents, agent) {
  if (ServiceModule.canActAsAgent(di) == true) {
    iAmAgent = true;
    newAgent = createsAgent(devicesSwarm, sequentialAgents, agent);
    for each device viz in myNeighbors
        accept = sendSubstitution(viz);
        if (accept) {
          notifyRotation(viz);
        }
    }
    stopUpdates(); // stop because agent responsibilities consumes energy
  }
  return newAgent; 
}
```


### Agent responsibilities

**[A1]** Search Process.
```javascript
a.SEARCH_LOCATION(Location loc, Device requesterDev, Agent requesterAg) {
  if (BordersModule.managesLocation(loc)) {
    if (ServiceModule.canAcceptNewDevices()) {
      notifyDevice(requesterDev);
      notifyAgent(reuesterAg, requesterDev);
      return a;
    } else {
      agent = SequentialModule.getSequentialAgent();
      return agent.SEARCH_LOCATION(loc, requesterDev, requesterAg);
    }
  } else {
    aNear = JumpingModule.getNearAgent(loc);
    ag = aNear.SEARCH_LOCATION(loc, requesterDev, requesterAg);
    if (amountRequests(loc) > MAX_REQS) {
      JumpingModule.updateRoutingTable(ag); // adds an entry in loc if agent ag accepted requesterDev
    }
  }
}

```

**[A2]** Monitor Process.
```javascript
a.MONITOR_SWARM() {
  frontier = getFrontierDevices();
  if (ServiceModule.hasTooManyDevices(frontier)) {
    for each pair devices di and dj in frontier {
      if (distance(di, dj) < deltaDistance) {
        purgeDeviceFromFrontier(dj); // to avoid scalability issues
      }
    }
  }
  for each device di in frontier {
    if (di.lastInformationSent() > deltaTime) {
      purgeDeviceFromSwarm(di);
    }
  }
}
```

**[A3]** Update Process.
```javascript
a.UPDATE_PROCESS(List deviceList) {
  for each device di in deviceList {
    location = di.location;
    frontierArea = BordersModule.updateFrontierArea(location)
  }
  sequentials = SequentialModule.getSequentialAgents();
  notifyNewFrontierArea(sequentials, frontierArea);
}
```

**[A4]** Merge Process.
```javascript
a.MERGE_SWARMS(List frontierAreas, Agent ar) { // ar is the agent responsible for FrontiersAreas
  myFrontierAreas = BordersModule.getFrontierAreas();
  if (intersects(myFrontierAreas, frontierAreas) {
    B = getBorderIntersection(myFrontierAreas, frontierAreas);
    fr = getFrontierDevices(ar, B);
    fa = getFrontierDevices(a, B);
    if (BordersModule.canMergeSwarms(fr, fa) && consensus(a, ar) == a) {
      swarm = getAllDevices(ar, B);
      frontierArea = BordersModule.updateFrontierArea(swarm);
      notifyNewFrontierArea(sequentials, frontierArea);
      updateSwarmEliminatingAgent(a, ar);
    }
  }
}
```

**[A5]** Split Process.
```javascript
a.SPLIT_PROCESS(Device di, Device dj) {
  loc1 = di.location;
  loc2 = dj.location;
  devicesPath = BorderModule.createsFrontierPath(loc1, loc2);
  if (devicesPath != null) { // the border was closed
    for each device di in devicePath {
      BorderModule.assignBorderResposibility(di);
    }
    BorderModule.updateFrontier(devicePath); // including the area
  } else {
    SPLIT_SWARM(di, dj);
  }
}
```

```javascript
a.SPLIT_SWARM(Device di, Device dj) {
  listDevA, listDevB  = BorderModule.separateDevices(di, dj); // separates the swarm into two devices list
  sequentials = SequentialModule.getSequentialAgents();
  di = chooseDevice(listDevA);
  newAgent = di.DEVICE_AGENTIFICATION(listDevA, sequentials, a);
  BorderModule.updateFrontier(listDevB);
  SequentialModule.addSequential(newAgent);
}
```


**[A6]** Divide Process.
```javascript
a.DIVIDE_PROCESS() {
  if (ServiceModule.hasTooManyDevices())
    frontierDevices = BordersModule.getFrontierDevices();
    di, dj = BordersModule.obtainSeparationDevices(); // di and dj are consecutive connected border devices
    SPLIT_SWARM(di, dj);
  }
  
}
```
