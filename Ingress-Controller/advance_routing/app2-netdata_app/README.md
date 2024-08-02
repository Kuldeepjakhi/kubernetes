
1. Create 3 Default storage class and pv for netdata.

`kubectl apply -f storage_class.yaml`  
`kubectl apply -f Pv_netdata.yaml`  

2. Clone netdata repository  

`git clone https://github.com/netdata/helmchart.git netdata-helmchart`

3. Install netdata using helm

`helm install netdata ./netdata-helmchart/charts/netdata`



