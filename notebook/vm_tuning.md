# 虛擬化的效能調校
[參考資料:http://dic.vbird.tw/network_project/unit04.php](http://dic.vbird.tw/network_project/unit04.php)

## XML 檔案的優化
### 1. 虛擬機器運作的考量
- 一般可接受的硬體配置：

類似大型的公有雲或者是私有雲的角度來看，這些雲系統內的 VM 為了可以在不同的硬體之間進行移轉 (live migration)， 因此，VM 的硬體配置必需要所有的 host 都可以運行才行！舉例來說，你的 VM 如果選擇了 Intel 的 I7 系列 CPU 來執行運作， 但是你卻將這個 VM 移動到 AMD 的 CPU 的 host 上面，這可能會產生一些比較嚴重的錯誤，導致 VM 無法運行。 以這種案例來說，你的 VM 硬體可能要使用大部分 host 都可以運作的配置為宜。不宜使用專屬於某個主機硬體的配置。
- 綁定特定主機資源的硬體配置：

以上面的案例來說，最好使用大部分 Host 都可以支援的硬體配置為宜，但是，如此一來，許多本機的 CPU 指令集可能就無法直接操作， 因而造成 VM 的效能與 host 的性能差異較大的情況。不過若以本課程的環境來說，其實我們的 VM 不太可能進行轉移 (雖然硬碟可以直接複製給其他 VM 使用)， 應該說，我們的 VM 不可能進行線上轉移 (live migration)，因此，若能使用本機的所有資源，將會有較好的 VM 效能。

### 2. CPU 的效能優化
- 其實 Intel 的 CPU 經常是 4 核 8 緒，但是 OS 都判定為 8 個 CPU 核心
- 本機的 CPU 情況，除了可以查看 /proc/cpuinfo 之外，也可以透過 cpupower 來觀察
```
cpupower monitor
------------------------------------------------------------------
    |Nehalem                    || SandyBridge        || Mperf              || Idle_Stats
CPU | C3   | C6   | PC3  | PC6  || C7   | PC2  | PC7  || C0   | Cx   | Freq || POLL | C1-S | C1E- | C3-S | C6-S
   0|  0.00| 99.80|  0.49| 98.65||  0.00|  0.07|  0.00||  0.00|100.00|  2970||  0.00|  0.00|  0.00|  0.00| 100.0
   4|  0.00| 99.80|  0.49| 98.65||  0.00|  0.07|  0.00||  0.13| 99.87|  3359||  0.00|  0.00|  0.04|  0.00| 99.82
   1|  0.07| 99.81|  0.49| 98.65||  0.00|  0.07|  0.00||  0.02| 99.98|  1905||  0.00|  0.00|  0.00|  0.07| 99.91
   5|  0.07| 99.82|  0.49| 98.65||  0.00|  0.07|  0.00||  0.00|100.00|  3633||  0.00|  0.00|  0.00|  0.00| 100.0
   2|  0.88| 98.98|  0.49| 98.65||  0.00|  0.07|  0.00||  0.01| 99.99|  2389||  0.00|  0.00|  0.00|  0.00|100.00
   6|  0.88| 98.98|  0.49| 98.65||  0.00|  0.07|  0.00||  0.01| 99.99|  2377||  0.00|  0.00|  0.00|  0.07| 99.93
   3|  0.01| 99.40|  0.49| 98.65||  0.00|  0.07|  0.00||  0.01| 99.99|  2413||  0.00|  0.00|  0.00|  0.00|100.00
   7|  0.01| 99.41|  0.49| 98.65||  0.00|  0.07|  0.00||  0.05| 99.95|  1736||  0.00|  0.00|  0.00|  0.00| 99.95
------------------------------------------------------------------
```
重點在看那個 Freq 的項目，就有不同的時脈說明。此外，你也可以查閱第一直行的 CPU ID 號碼，你就可以發現， 0, 4 放在一起， 1, 5 放在一起，亦即這幾個 ID 使用的是同一個核心的意思。
- 事實上，libvirtd 也能自動的找到正確的 CPU 對應！你可以使用底下的指令來觀察
```
# 列出 BIOS、主機板製造商、CPU型號、每個插槽的記憶體資訊等
virsh sysinfo 
------------------------------------------------------------------
<sysinfo type='smbios'>
  <bios>
    <entry name='vendor'>American Megatrends Inc.</entry>
    <entry name='version'>1001</entry>
    <entry name='date'>06/28/2012</entry>
    <entry name='release'>4.6</entry>
  </bios>
  <system>
    <entry name='manufacturer'>ASUSTeK Computer INC.</entry>
    <entry name='product'>BM6660(BM6360)</entry>
    <entry name='version'>System Version</entry>
    <entry name='serial'>C9PFAG0019SB</entry>
    <entry name='uuid'>c3017fa0-030c-11e2-b07d-3085a9a75a09</entry>
    <entry name='sku'>SKU</entry>
    <entry name='family'>To be filled by O.E.M.</entry>
  </system>
  <baseBoard>
    <entry name='manufacturer'>ASUSTeK COMPUTER INC.</entry>
    <entry name='product'>BM6660(BM6360)</entry>
    <entry name='version'>Rev 1.xx</entry>
    <entry name='serial'>120902045401312</entry>
    <entry name='asset'>To be filled by O.E.M.</entry>
    <entry name='location'>To be filled by O.E.M.</entry>
  </baseBoard>
  <chassis>
    <entry name='manufacturer'>Chassis Manufacture</entry>
    <entry name='version'>Chassis Version</entry>
    <entry name='serial'>Chassis Serial Number</entry>
  </chassis>
  <processor>
    <entry name='socket_destination'>LGA1155</entry>
    <entry name='type'>Central Processor</entry>
    <entry name='family'>Core i7</entry>
    <entry name='manufacturer'>Intel</entry>
    <entry name='signature'>Type 0, Family 6, Model 42, Stepping 7</entry>
    <entry name='version'>Intel(R) Core(TM) i7-2600 CPU @ 3.40GHz</entry>
    <entry name='external_clock'>100 MHz</entry>
    <entry name='max_speed'>3800 MHz</entry>
    <entry name='status'>Populated, Enabled</entry>
    <entry name='serial_number'>To Be Filled By O.E.M.</entry>
    <entry name='part_number'>To Be Filled By O.E.M.</entry>
  </processor>
  <memory_device>
    <entry name='size'>4096 MB</entry>
    <entry name='form_factor'>DIMM</entry>
    <entry name='locator'>DIMM0</entry>
    <entry name='bank_locator'>BANK0</entry>
    <entry name='type'>DDR3</entry>
    <entry name='type_detail'>Synchronous</entry>
    <entry name='speed'>1333 MT/s</entry>
    <entry name='manufacturer'>Undefined</entry>
    <entry name='serial_number'>0000010</entry>
    <entry name='part_number'>SLA302G08-GGNNG</entry>
  </memory_device>
  <memory_device>
    <entry name='size'>2048 MB</entry>
    <entry name='form_factor'>DIMM</entry>
    <entry name='locator'>DIMM1</entry>
    <entry name='bank_locator'>BANK1</entry>
    <entry name='type'>DDR3</entry>
    <entry name='type_detail'>Synchronous</entry>
    <entry name='speed'>1333 MT/s</entry>
    <entry name='manufacturer'>Undefined</entry>
    <entry name='serial_number'>0000E40</entry>
    <entry name='part_number'>SLZ302G08-MDJHB</entry>
  </memory_device>
  <memory_device>
    <entry name='size'>4096 MB</entry>
    <entry name='form_factor'>DIMM</entry>
    <entry name='locator'>DIMM2</entry>
    <entry name='bank_locator'>BANK2</entry>
    <entry name='type'>DDR3</entry>
    <entry name='type_detail'>Synchronous</entry>
    <entry name='speed'>1333 MT/s</entry>
    <entry name='manufacturer'>Undefined</entry>
    <entry name='serial_number'>0000012</entry>
    <entry name='part_number'>SLA302G08-GGNNG</entry>
  </memory_device>
  <memory_device>
    <entry name='size'>2048 MB</entry>
    <entry name='form_factor'>DIMM</entry>
    <entry name='locator'>DIMM3</entry>
    <entry name='bank_locator'>BANK3</entry>
    <entry name='type'>DDR3</entry>
    <entry name='type_detail'>Synchronous</entry>
    <entry name='speed'>1333 MT/s</entry>
    <entry name='manufacturer'>Undefined</entry>
    <entry name='serial_number'>0000C00</entry>
    <entry name='part_number'>SLZ302G08-MDJHB</entry>
  </memory_device>
</sysinfo>
------------------------------------------------------------------

virsh capabilities 
------------------------------------------------------------------
<capabilities>

  <host>
    <uuid>c3017fa0-030c-11e2-b07d-3085a9a75a09</uuid>
    <cpu>
      <arch>x86_64</arch>
      <model>SandyBridge-IBRS</model>
      <vendor>Intel</vendor>
      <microcode version='46'/>
      <topology sockets='1' cores='4' threads='2'/>
      <feature name='vme'/>
      <feature name='ds'/>
      <feature name='acpi'/>
      <feature name='ss'/>
      <feature name='ht'/>
      <feature name='tm'/>
      <feature name='pbe'/>
      <feature name='dtes64'/>
      <feature name='monitor'/>
      <feature name='ds_cpl'/>
      <feature name='vmx'/>
      <feature name='smx'/>
      <feature name='est'/>
      <feature name='tm2'/>
      <feature name='xtpr'/>
      <feature name='pdcm'/>
      <feature name='pcid'/>
      <feature name='osxsave'/>
      <feature name='arat'/>
      <feature name='stibp'/>
      <feature name='ssbd'/>
      <feature name='xsaveopt'/>
      <feature name='invtsc'/>
      <pages unit='KiB' size='4'/>
      <pages unit='KiB' size='2048'/>
    </cpu>
    <power_management>
      <suspend_mem/>
      <suspend_disk/>
      <suspend_hybrid/>
    </power_management>
    <iommu support='no'/>
    <migration_features>
      <live/>
      <uri_transports>
        <uri_transport>tcp</uri_transport>
        <uri_transport>rdma</uri_transport>
      </uri_transports>
    </migration_features>
    <topology>
      <cells num='1'>
        <cell id='0'>
          <memory unit='KiB'>12526728</memory>
          <pages unit='KiB' size='4'>3131682</pages>
          <pages unit='KiB' size='2048'>0</pages>
          <distances>
            <sibling id='0' value='10'/>
          </distances>
          <cpus num='8'>
            <cpu id='0' socket_id='0' core_id='0' siblings='0,4'/>
            <cpu id='1' socket_id='0' core_id='1' siblings='1,5'/>
            <cpu id='2' socket_id='0' core_id='2' siblings='2,6'/>
            <cpu id='3' socket_id='0' core_id='3' siblings='3,7'/>
            <cpu id='4' socket_id='0' core_id='0' siblings='0,4'/>
            <cpu id='5' socket_id='0' core_id='1' siblings='1,5'/>
            <cpu id='6' socket_id='0' core_id='2' siblings='2,6'/>
            <cpu id='7' socket_id='0' core_id='3' siblings='3,7'/>
          </cpus>
        </cell>
      </cells>
    </topology>
    <cache>
      <bank id='0' level='3' type='both' size='8' unit='MiB' cpus='0-7'/>
    </cache>
    <secmodel>
      <model>selinux</model>
      <doi>0</doi>
      <baselabel type='kvm'>system_u:system_r:svirt_t:s0</baselabel>
      <baselabel type='qemu'>system_u:system_r:svirt_tcg_t:s0</baselabel>
    </secmodel>
    <secmodel>
      <model>dac</model>
      <doi>0</doi>
      <baselabel type='kvm'>+107:+107</baselabel>
      <baselabel type='qemu'>+107:+107</baselabel>
    </secmodel>
  </host>

  <guest>
    <os_type>hvm</os_type>
    <arch name='i686'>
      <wordsize>32</wordsize>
      <emulator>/usr/libexec/qemu-kvm</emulator>
      <machine maxCpus='240'>pc-i440fx-rhel7.0.0</machine>
      <machine canonical='pc-i440fx-rhel7.0.0' maxCpus='240'>pc</machine>
      <machine maxCpus='240'>rhel6.0.0</machine>
      <machine maxCpus='240'>rhel6.1.0</machine>
      <machine maxCpus='240'>rhel6.2.0</machine>
      <machine maxCpus='240'>rhel6.3.0</machine>
      <machine maxCpus='240'>rhel6.4.0</machine>
      <machine maxCpus='240'>rhel6.5.0</machine>
      <machine maxCpus='240'>rhel6.6.0</machine>
      <domain type='qemu'/>
      <domain type='kvm'>
        <emulator>/usr/libexec/qemu-kvm</emulator>
      </domain>
    </arch>
    <features>
      <cpuselection/>
      <deviceboot/>
      <disksnapshot default='off' toggle='no'/>
      <acpi default='on' toggle='yes'/>
      <apic default='on' toggle='no'/>
      <pae/>
      <nonpae/>
    </features>
  </guest>

  <guest>
    <os_type>hvm</os_type>
    <arch name='x86_64'>
      <wordsize>64</wordsize>
      <emulator>/usr/libexec/qemu-kvm</emulator>
      <machine maxCpus='240'>pc-i440fx-rhel7.0.0</machine>
      <machine canonical='pc-i440fx-rhel7.0.0' maxCpus='240'>pc</machine>
      <machine maxCpus='240'>rhel6.0.0</machine>
      <machine maxCpus='240'>rhel6.1.0</machine>
      <machine maxCpus='240'>rhel6.2.0</machine>
      <machine maxCpus='240'>rhel6.3.0</machine>
      <machine maxCpus='240'>rhel6.4.0</machine>
      <machine maxCpus='240'>rhel6.5.0</machine>
      <machine maxCpus='240'>rhel6.6.0</machine>
      <domain type='qemu'/>
      <domain type='kvm'>
        <emulator>/usr/libexec/qemu-kvm</emulator>
      </domain>
    </arch>
    <features>
      <cpuselection/>
      <deviceboot/>
      <disksnapshot default='off' toggle='no'/>
      <acpi default='on' toggle='yes'/>
      <apic default='on' toggle='no'/>
    </features>
  </guest>

</capabilities>
------------------------------------------------------------------
```
- 因為如果一個 VM 使用到的 CPU 是同一個運算核心，那麼當 VM 系統很忙碌的時候， 就可能會造成只使用到單顆 CPU 核心的困境～並不是兩顆喔！所以，最佳的情況，就是將 VM 的 CPU 綁定在不同的核心上， 會比較能夠榨出效能。這時需要修改 XML 的設定，處理一下
```
vim /vmdisk/centos7.xml
------------------------------------------------------------------
# 原本是這樣：
  <vcpu>4</vcpu>
  ....
  <cpu mode="host-model"/>

# 可以改成這樣：
  <vcpu placement='static'>4</vcpu>
  <cputune>
     <vcpupin vcpu='0' cpuset='4'/>    # 分別對應在不同的運算核心上面，使用 4~7 或 0~3 均可！
     <vcpupin vcpu='1' cpuset='5'/>
     <vcpupin vcpu='2' cpuset='6'/>
     <vcpupin vcpu='3' cpuset='7'/>
  </cputune>
  <cpu mode='host-model'>                         # 將指令 bypass 給 host CPU！
    <topology sockets='1' cores='4' threads='1'/> # 使用與 host 相同的設計，只是 threads 設計為 1 個
  </cpu>
------------------------------------------------------------------
```
-  一般來說，系統為了可以讓 CPU 均勻的被使用，因此會啟動一個名為 irqbalance 的服務， 這個服務會自動的將各個工作在各 CPU 之間轉來轉去。因此，如果你希望可以綁定 VM 的 CPU 對應到實體 CPU 的 ID 上面， 最好將這個 irqbalance 的服務關閉比較妥當！否則，可能你的設定不會有效果的！
```
systemctl stop irqbalance.service
systemctl disable irqbalance.service
```
- 關於 numad 這個服務，一般來說，指令動作在同一個 cpu 插槽運作時，效能會比較好！ 但是，有時候 CPU 數量不足，因此，主機板開發商有時會透過 CPU 製造商的設計，建立兩顆以上的 CPU 插槽在一個主機板上面， 因此有所謂的二路、四路 (兩顆、四顆) 的 CPU 主機板。但是，如此一來，某些硬體的線路配置，就得要單獨設計到不同的 CPU 上面！ 這時，如果能夠分配工作到正確的 CPU ID 上，將會有助於一點點的效能提昇
```
# 觀察每顆 CPU 核心對應的工作 (IRQ)
           CPU0       CPU1       CPU2       CPU3       CPU4       CPU5       CPU6       CPU7
  0:         37          0          0          0          0          0          0          0   IO-APIC-edge      timer
  1:          5          0          0          0          0          0          0          0   IO-APIC-edge      i8042
  8:        271          0          0          0          0          0          0          0   IO-APIC-edge      rtc0
  9:          4          0          0          0          0          0          0          0   IO-APIC-fasteoi   acpi
 12:          6          0          0          0          0          0          0          0   IO-APIC-edge      i8042
 18:          0          0          0          0          0          0          0          0   IO-APIC-fasteoi   i801_smbus
 23:         64          0          0          0          0          0          0          0   IO-APIC-fasteoi   ehci_hcd:usb1, ehci_hcd:usb2
 24:          0          0          0          0          0          0          0          0   PCI-MSI-edge      PCIe PME
 25:          0          0          0          0          0          0          0          0   PCI-MSI-edge      PCIe PME
 26:          0          0          0          0          0          0          0          0   PCI-MSI-edge      PCIe PME
 27:          0          0          0          0          0          0          0          0   PCI-MSI-edge      xhci_hcd
 28:          0          0          0          0          0          0          0          0   PCI-MSI-edge      xhci_hcd
 29:          0          0          0          0          0          0          0          0   PCI-MSI-edge      xhci_hcd
 30:          0          0          0          0          0          0          0          0   PCI-MSI-edge      xhci_hcd
 31:          0          0          0          0          0          0          0          0   PCI-MSI-edge      xhci_hcd
 32:          0          0          0          0          0          0          0          0   PCI-MSI-edge      xhci_hcd
 33:          0          0          0          0          0          0          0          0   PCI-MSI-edge      xhci_hcd
 34:          0          0          0          0          0          0          0          0   PCI-MSI-edge      xhci_hcd
 35:        119          0       1364          0          0          0          0    7637242   PCI-MSI-edge      eno1
 36:    7817900          0          0          4          0          0          0          0   PCI-MSI-edge      0000:00:1f.2
 37:         53          0          0          0          0          0          0          0   PCI-MSI-edge      i915
 38:        273          0          0          0          0          0          0          0   PCI-MSI-edge      snd_hda_intel:card0
NMI:         54         46         21         22         14          7          8         26   Non-maskable interrupts
LOC:    2367064    2094631    1868359    1352873     664718     606155     620912    2457331   Local timer interrupts
SPU:          0          0          0          0          0          0          0          0   Spurious interrupts
PMI:         54         46         21         22         14          7          8         26   Performance monitoring interrupts
IWI:      24306      44610      50243      38363       6114       6762       4909     352165   IRQ work interrupts
RTR:          0          0          0          0          0          0          0          0   APIC ICR read retries
RES:     234664       5301       7128      11700      22904      16328      13042      15614   Rescheduling interrupts
CAL:       1876       1676       1842       1806       1781       1781       1830       1755   Function call interrupts
TLB:        159        236        217        138       6870       6142       6861       3868   TLB shootdowns
TRM:          0          0          0          0          0          0          0          0   Thermal event interrupts
THR:          0          0          0          0          0          0          0          0   Threshold APIC interrupts
DFR:          0          0          0          0          0          0          0          0   Deferred Error APIC interrupts
MCE:          0          0          0          0          0          0          0          0   Machine check exceptions
MCP:       1990       1990       1990       1990       1990       1990       1990       1990   Machine check polls
ERR:          0
MIS:          0
PIN:          0          0          0          0          0          0          0          0   Posted-interrupt notification event
NPI:          0          0          0          0          0          0          0          0   Nested posted-interrupt event
PIW:          0          0          0          0          0          0          0          0   Posted-interrupt wakeup event
```
我們可以看見 eno1 這個網路卡以及 0000:00:1f.2 這個元件最忙碌！eno1 我們已經知道是網路卡，那麼 0000:00:1f.2 是什麼鬼？ 我們可以透過 lspci 來查詢主機板的硬體線路配置：
```
lspci
------------------------------------------------------------------
00:00.0 Host bridge: Intel Corporation 2nd Generation Core Processor Family DRAM Controller (rev 09)
00:01.0 PCI bridge: Intel Corporation Xeon E3-1200/2nd Generation Core Processor Family PCI Express Root Port (rev 09)
00:02.0 VGA compatible controller: Intel Corporation 2nd Generation Core Processor Family Integrated Graphics Controller (rev 09)
00:19.0 Ethernet controller: Intel Corporation 82579LM Gigabit Network Connection (Lewisville) (rev 05)
00:1a.0 USB controller: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #2 (rev 05)
00:1b.0 Audio device: Intel Corporation 6 Series/C200 Series Chipset Family High Definition Audio Controller (rev 05)
00:1c.0 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 1 (rev b5)
00:1c.7 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 8 (rev b5)
00:1d.0 USB controller: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #1 (rev 05)
00:1e.0 PCI bridge: Intel Corporation 82801 PCI Bridge (rev a5)
00:1f.0 ISA bridge: Intel Corporation Q67 Express Chipset LPC Controller (rev 05)
00:1f.2 RAID bus controller: Intel Corporation SATA Controller [RAID mode] (rev 05)
00:1f.3 SMBus: Intel Corporation 6 Series/C200 Series Chipset Family SMBus Controller (rev 05)
03:00.0 USB controller: NEC Corporation uPD720200 USB 3.0 Host Controller (rev 04)
------------------------------------------------------------------
```
果然是硬碟所在處！所以忙碌的是網路與磁碟的 I/O 喔！好在他們分配的 IRQ 對應的 CPU 不一樣！還好！ 但是未來 4~7 都要分配給 VM 來運作，同時 CPU0 經常會負責一些突發的動作，因此假設你想要讓 eno1 分配在 CPU2 ， 而 RAID 想要配置到 CPU3 時，該怎麼進行呢？這需要使用 2 進位的方式來考慮！：
```
CPU7 CPU6 CPU5 CPU4 CPU3 CPU2 CPU1 CPU0    10進位     16進位
   1    1    1    1    1    1    1    1 -> 255     -> ff
```
10 進位算法很單純，那 16 進位呢？很簡單，因為 24 就是 16 ，因此， 4 個 2 進位放在一起， 所以就變成兩組～結果就是 2 進位 (1111)(1111) 變成 10 進位 (15)(15)，轉成 16 進位表示，就會變成 (f)(f) 的結果了。 那麼每個 IRQ 的 CPU 設定在哪裡呢？如上頭的 /proc/interrupts 的內容，我們知道 irq 35 是 eno1， 所以來看看這個 35 號 IRQ 的 CPU 列表：
```
cat /proc/irq/35/smp_affinity
80
cat /proc/irq/36/smp_affinity
01
```
上面顯示的結果是 16 進位喔！根據 16 進位的結果， 80 會變成 (8)(0) 轉成 2 進位就會變成 (1000)(0000)，因此只有 CPU7 會操作 IRQ 35！ 至於 01 就會變成 (0)(1) 也就是 (0000)(0001)，亦即僅有 CPU0 會負責這個 36 號 IRQ 的運作。好了！那麼讓 35 變成 (0000)(0100) 而 36 變成 (0000)(1000) 時，可以分別得到 35 IRQ 應該是 04 而 36 IRQ 應該是 08 才對！因此我們就這麼做：
```
echo 04 > /proc/irq/35/smp_affinity
echo 08 > /proc/irq/36/smp_affinity
```
有時你可能需要執行兩次才會生效！處理完畢再重新去看一下 /proc/interrupts 的內容，就會發現不一樣了！不過，這樣的動作對單 CPU 插槽來說， 差異沒有很大！但是如果是很忙的系統，你想要讓系統的資料分流，這可能也是一個好方案！同時，如果系統有多個 CPU 插槽， 請自行參考主機板製造商提供的線路說明，例如 [Supermicro X10DRi 主機板](https://www.supermicro.com/products/motherboard/xeon/c600/x10dri.cfm)，你會看到有兩顆 CPU 插槽，而在其 [說明手冊內頁 (page 1-8)](https://www.supermicro.com/manuals/motherboard/C606_602/MNL-1491.pdf) 的 CPU 與所有週邊設備的運作中，畫面左側的 CPU 是 CPU1 而右側是 CPU2， 所以，你就應該可以理解，應該要將不同的 PCI-E 界面安插在那一個上面，其與 CPU 的溝通中，又應該以誰為優先考量，這就值得去設計了！
- 使用 numactl 來規範某個程式要指定哪顆 CPU 運作。這也是挺有趣的！ 你可以先透過 ps -eF 找出某個 PID 對應的 CPU 號碼，例如
```
yum install numactl

numactl -H
------------------------------------------------------------------

available: 1 nodes (0)
node 0 cpus: 0 1 2 3 4 5 6 7
node 0 size: 12233 MB
node 0 free: 8589 MB
node distances:
node   0
  0:  10
------------------------------------------------------------------

ps -eF
------------------------------------------------------------------
UID        PID  PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
root         1     0  0 48479  7136   0  3月12 ?      00:00:12 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root         2     0  0     0     0   5  3月12 ?      00:00:00 [kthreadd]
root         3     2  0     0     0   0  3月12 ?      00:00:00 [ksoftirqd/0]
root         5     2  0     0     0   0  3月12 ?      00:00:00 [kworker/0:0H]
------------------------------------------------------------------
```
之後再以 taskset 來更新 PID 對應的 CPU 號碼即可：
```
taskset -cp cpuN PID
```
例如重複執行 PI 的計算！
```
time echo "scale=10000;4*a(1)" | numactl -C 1 bc -lq &> /dev/null &
top -d 2  # (按下 1 觀察個別 CPU 的變化)
------------------------------------------------------------------
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
22223 root      20   0   13364   1116    716 R  33.3  0.0   0:13.84 bc
22217 root      20   0   13364   1120    716 R  26.7  0.0   0:15.34 bc
22220 root      20   0   13364   1112    716 R  26.7  0.0   0:14.28 bc
22226 root      20   0   13364   1112    716 R  20.0  0.0   0:13.53 bc
------------------------------------------------------------------
```
然後透過 taskset 來改變不同的 PID 到不同的 CPU 去
```
taskset -cp 2 22226
```
透過這樣的設計，你也可以將 VM 的 CPU 全部綁在同一個 CPU 上面～呵呵！那就好笑了！因為， VM 雖然看到是 4 顆 CPU， 但是其實只用到的實體系統的一個 CPU ID 而已喔！
- 如果擔心 VM 的 CPU 腳位設定有問題，可以透過底下的方式來處理觀察喔
```
VCPU：         0
處理器：    4
狀態：       執行中
處理器時間： 1819.4s
處理器的同屬： ----y---  # <==重點就是這個東西囉！

VCPU：         1
處理器：    5
狀態：       執行中
處理器時間： 1133.0s
處理器的同屬： -----y--

VCPU：         2
處理器：    6
狀態：       執行中
處理器時間： 1087.7s
處理器的同屬： ------y-

VCPU：         3
處理器：    7
狀態：       執行中
處理器時間： 962.2s
處理器的同屬： -------y
```
如果要修改，就得要使用底下的方式來處理：
```
virsh vcpuping domainN VcpuN cpuN
```

### 3. 磁碟性能調校
基本上，當初使用 virt-install 時，我們已經加入了許多磁碟 I/O 的效能優化調整，這部份就稍微能夠省略的！ 除非未來有問題，再回來進行微調～否則目前包括 cache 以及 io 的設計，應該能符合大部分的使用需求了！
```
vim /vmdisk/centos7.xml
------------------------------------------------------------------
    <disk type="file" device="disk">
      <driver name="qemu" type="qcow2" cache="writeback" io="threads"/>
      <source file="/vmdisk/centos7.raw.img"/>
      <target dev="vda" bus="virtio"/>
    </disk>
------------------------------------------------------------------
```

### 4. 顯示卡調校
- 我們大部分使用文字界面來操作系統，所以這個部份顯的不是很重要！但是，如果你需要圖形界面時，或許這個設定還需要調整一下比較好。 我們在安裝時，已經指定了 qxl 這個顯示卡界面，但是並沒有指定顯示卡的記憶體相關資訊，所以，這裡我們可以做點變化
```
vim /vmdisk/centos7.xml
------------------------------------------------------------------
# 原本是這樣
    <video>
      <model type="qxl"/>
    </video>

# 嘗試改成這樣：
    <video>
      <model type="qxl" vram64='16384' heads='1' />
    </video>
------------------------------------------------------------------
```
- 除了顯示卡本身之外，連線的方式也可以進行效能調整！可以找到 graphical 的項目來調整這個連線的狀態！ 如果是考量傳輸資料的頻寬，那麼傳輸過程中，所有可以進行影像壓縮的功能，全部都可以啟動！如此則可以減少頻寬的使用
```
vim /vmdisk/centos7.xml
------------------------------------------------------------------
# 原本長這樣：
    <graphics type="spice" port="5911" listen="0.0.0.0" passwd="xxxxxxxxxxx">
      <image compression="off"/>
    </graphics>

# 可以改成這個樣子：
    <graphics type="spice" port="5911" listen="0.0.0.0" passwd="xxxxxxxxxxx">
       <image compression='auto_glz' />
       <jpeg compression='auto' />
       <zlib compression='auto' />
       <playback compression='on' />
       <streaming mode='filter' />
    </graphics>
------------------------------------------------------------------
```
 - 最後，如果沒有需要用到 spice 的 USB 傳輸偵測，那麼可以將 spice 相關的 channel 取消， 這樣可以讓虛擬終端機的運作較為快速！免得一直在 server / client 之間偵測 usb 的行為，導致傳輸有點慢
```
vim /vmdisk/centos7.xml
# 找到底下這些關鍵字：
    <channel type="unix">
      <source mode="bind"/>
      <target type="virtio" name="org.qemu.guest_agent.0"/>
    </channel>
    <channel type="spicevmc">
      <target type="virtio" name="com.redhat.spice.0"/>
    </channel>
    <redirdev bus="usb" type="spicevmc"/>
    <redirdev bus="usb" type="spicevmc"/>
# 通通刪除囉！
```
### 5. 其實最好調整好一個，就處理一次重新啟動的任務，方便找出哪邊有不支援的參數，修改上面會比較快速！ 全部都處理完畢之後，就可以完整關閉虛擬機器，然後再重新啟動虛擬機器，這樣才能夠讓 XML 檔案的內容生效！

```
