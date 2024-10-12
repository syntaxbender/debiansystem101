```bash
$ guestfish --cmd-help #tüm command listesi
$ guestfish --rw -a 2024-07-04-raspios-bookworm-arm64-lite.img
> add disk.img # -a argümanıyla belirttiğim diskl imajını aynı zamanda interactive shell ile de belirtebiliyorum.
> run #imajı bağlar
> list-partitions #imajdaki partitionları listeler
> mount /dev/sda1 / #imajdaki /dev/sda1 partitionu / dizinine mount eder.
> ls / #mount edilen dizinde partitionu listeler.
> help {COMMAND} #belirtilen COMMAND'ın helpini listeler.
```

```bash
# non-interactive shell ile çalışıyorsam örnek bir komut;

$ guestfish add $IMAGE_DIR/$DISTRO_FILE : \
  run : \ #imajı bağlar
  mount /dev/sda1 / : \ #imajdaki /dev/sda1 partitionu / dizinine mount eder.
  copy-out / /mnt/boot : \ #mount edilen partitiondaki dosyaları "/mnt/boot" dizinine kopyalar.
  umount / : \ # /dev/sda1'i umount yapar. 
  mount /dev/sda2 / : \ #imajdaki /dev/sda2 partitionu / dizinine mount eder.
  copy-out / /mnt/root #mount edilen partitiondaki dosyaları "/mnt/root" dizinine kopyalar.

# non-interactive bir başka örnek;

$ guestfish -N $BUILD_DIR/distro.img=bootroot:vfat:ext4:2G # yeni bir disk imajı oluşturmak üzere komut girilir.
# boot ve root'tan oluşan 2 partitiona sahip bir disk imajı oluşturur.
# boot partititon filesystem'i vfat, root partition ise ex4 olarak ayarlanır.
# boot partition default olarak yanlış hatırlamıyorsam 4mb, 2G-4mb ise root partition boyutu olarak ayarlanır.

$ guestfish add $BUILD_DIR/distro.img : \ # bu örnekte de oluşturduğumuz boş imaja boot ve root dosyalarını kopyalıyoruz.
  run : \ #imajı bağlar 
  mount /dev/sda1 / : \ # mount eder 
  glob copy-in /mnt/boot/* / : \ # "/mnt/boot/" dizini içerisindeki dosyaları mount edilen dizine kopyalar. buradaki "glob" flagı ise wildcard(dizindeki tüm dosyaları seçen "*" karakteri için) kullanımı için gereklidir.
  umount / : \ # umount
  mount /dev/sda2 / : \ # mount eder
  glob copy-in /mnt/root/* / # "/mnt/root/" dizini içerisindeki dosyaları mount edilen dizine kopyalar. buradaki "glob" flagı ise wildcard(dizindeki tüm dosyaları seçen "*" karakteri için) kullanımı için gereklidir.


#### BONUS ####
$ sfdisk --part-type $BUILD_DIR/distro.img 1 c # boot partition üzerinde fs değişikliği yapıyorum. "--part-type" ile partition türünü değiştireceğimi ifade ediyorum. 1 ise imajdaki boot partitionu işaret eder. "c" ise bir, bölüm türü kodudur. c kodu W95 FAT32 (genellikle flash bellek gibi aygıtlar için kullanılan bir dosya sistemi) için kullanılır.
```
