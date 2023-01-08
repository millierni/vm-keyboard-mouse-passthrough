# Virtual Machine - Keyboard & Mouse Passthrough
## VIRTUAL MACHINE - KEYBOARD & MOUSE PASSTHROUGH [evdev]

- Find the keyboard and mouse device id
  ```
  ls /dev/input/by-id
  ```
- Test to know if it's the right keyboard
  - Replace `{keyboard_device_id}` by your keyboard device id
  ```
  sudo cat /dev/input/by-id/{keyboard_device_id}
  ```
- Edit your virtual machine config
  - Replace `win10` by your vm name
  ```
  sudo EDITOR=nano virsh edit win10
  ```
  - Replace the first line with
  ```
  <domain type='kvm' id='1' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
  ```
  - Add at the bottom, just before `</domain>`  
  Replace `{mouse_device_id}` and `{keyboard_device_id}` by your mouse & keyboard device id
  ```
  <qemu:commandline>
   <qemu:arg value='-object'/>
   <qemu:arg value='input-linux,id=mouse1,evdev=/dev/input/by-id/{mouse_device_id}'/>
   <qemu:arg value='-object'/>
   <qemu:arg value='input-linux,id=kbd1,evdev=/dev/input/by-id/{keyboard_device_id},grab_all=on,repeat=on'/>
  </qemu:commandline>
  ```
  - Add just before `<input type='mouse' bus='ps2'/>` and `<input type='keyboard' bus='ps2'/>`
  ```
  <input type='mouse' bus='virtio'>
          <address type='pci' domain='0x0000' bus='0x00' slot='0x0e' function='0x0'/>
  </input>
  <input type='keyboard' bus='virtio'>
          <address type='pci' domain='0x0000' bus='0x00' slot='0x0f' function='0x0'/>
  </input>
  ```
- Edit `qemu.conf`
  ```
  sudo nano /etc/libvirt/qemu.conf
  ```
  - Find and edit `cgroup_device_acl` to looks like
  ```
  cgroup_device_acl = [
          "/dev/null", "/dev/full", "/dev/zero", 
          "/dev/random", "/dev/urandom",
          "/dev/ptmx", "/dev/kvm", "/dev/kqemu",
          "/dev/rtc","/dev/hpet",
          "/dev/input/by-id/KEYBOARD_NAME",
          "/dev/input/by-id/MOUSE_NAME"
  ]
  ```

  - Find and uncomment `user = "root"`
  ```
  #user = "root"
  user = "root"
  ```
- Restart `libvirtd`
  ```
  systemctl restart libvirtd
  ```
