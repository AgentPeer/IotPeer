## Swim Traces

Each trace_X;...tar.gz has traces of X people, simulated using SWIM (Small World In Motion) http://swim.di.uniroma1.it/.

* The parameter used to generate the traces were:

Nodes                    X

NodeRadius               0.35

KnowingTime              10

SimulationSeconds        120

CellDistanceWeight       0.85

NodeSpeedMultiplier      1

WaitingTimeExponent      1.35

WaitingTimeUpperBound    10


* The format of each trace is: 

    Second Millisecond PersonId positionX positionY

Where PersonId is an unique identifier for the Person.
