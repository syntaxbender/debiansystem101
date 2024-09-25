# Shrinking images on Linux(Resize .img filesystems)
```
sudo modprobe loop # loop dev'i aktive eder
sudo losetup -f # yeni bir loopback dev'i tahsis eder. çıktıda tahsis edilen dev döner.
sudo losetup /dev/loop0 myimage.img # imajı dev'e bağlar.
sudo partprobe /dev/loop0 # imajı bağladıktan sonra kernel'de belirtilen dev'in partition table update atar. partitiona erişim sağlanır.
sudo gparted /dev/loop0 # gparted
```

https://softwarebakery.com//shrinking-images-on-linux
