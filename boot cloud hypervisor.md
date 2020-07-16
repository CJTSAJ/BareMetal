## Reference
https://blog.csdn.net/nirendao/article/details/76451491

## set ubuntu cloud image password
sudo apt install libguestfs-tools
sudo virt-customize -a image_name.raw --root-password password:qwwer1234
