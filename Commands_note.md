-For classwork
```
sinteractive --reservation=aneq505 --time=01:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
```

-For HW
```
ainteractive --reservation=aneq505 --time=01:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
```
-Navigate into the folder
```
cd decomp_tutorial
```


-Remove any unwanted stuff lingering around
```
module purge
```

Load qiime
```
module load qiime2/2024.10_amplicon
```

