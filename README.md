Bitcoin POS
===========
The idea was to build a simple all-in-one solution for small businesses wanting to accept bitcoin. We use cheap, easily available components to build the minimal setup necessary to accept bitcoins at a register.

The cashier types payment total on a numpad, a QR code is generated for the customer to scan, and a confirmation message is displayed when the payment is received.

The device is a numpad with a 2x16 character LCD display that a cashier interacts with, and a small LCD display facing the customer. 

We have a short Youtube video demonstrating its usage: http://youtu.be/3uetgK0brgA

The typical interaction in a business:
--------------------------------------
* Cashier asks how the customer will pay
* Customer replies ‘In bitcoin’ ;)
* Cashier enters total in to the device via numberpad, total appears on 2x16 character display
* Cashier presses enter
* QR code and dollar and btc amount appear on LCD in front of customer
* Customer scans QR code which initiates bitcoin transaction (using blockchain or other app)
* Device displays confirmation message on LCD and character display upon bitcoin transaction confirmation (uses the unconfirmed transaction confirmation)
* The payment is optionally sent to a forwarding address

Cashier Manual:
---------------
* The display shows "Payment Amount". Type in total amount or calculate total then press enter.
* The display shows "Payment Entered" followed by "QR Code Ready" when the QR code is ready for the customer.
* If you need to cancel the payment, you can press Backspace three times in a row to cancel the transaction. The display will say "Payment Canceled", press enter to clear and begin a new transaction.
* Once the transaction is confirmed, the display shows "Transaction Approved".

CSV log:
--------
Set file name in run.py to enable transaction log. The format is:
(datetime,success,amount $,BTC address,BTC amount)

*Datetime* transaction timestamp in ISO format YYYY-mm-dd HH:MM:SS . Uses local system clock, depends 
on NTP synchronization and time zone being set up correctly.

*Success* One of following values:
* OK if payment went through. Filter on this if you want to see only successful 
  payments.
* CANC if payment was cancelled by operator
* BAD if the amount was too big (for whatever limit set by Blockchain.info) or 
  too small - less than 0.0001 BTC, to avoid problems with transaction fees.

*BTC address* is address the customer paid to, not the forwarding address. Useful 
for troubleshooting.

The Materials Used:
-------------------
* Raspberry Pi
* 3.2” LCD RCA output Monitor
* 2x16 character LED display
* USB numpad
* 12v 1a power supply
* Raspberry Pi enclosure
* 5v step down regulator (ripped from phone car-charger)
* micro-usb adapter male tip
* RCA male-to-male adapter or RCA cable

The Codebase:
-------------
* Raspbian
* Python
* Python-requests
   - web requests
* Python-qrencode
* 2x16 character library
* tkInter - graphics
* Blockchain.info API
   - create new bitcoin addresses
   - get the BTC/USD exchange rate
   - send BTC to forwarding address

Steps to Build Device:
----------------------
* attach 2x16 character display to Raspberry Pi
* solder together the 3.2” display’s power wires, power supply wires, and 5v step-down converter’s input wires positive to positive to positive and  negative to negative to negative.
* solder together the micro-USB wires to the 5v step-down converter’s output wires positive to positive and negative to negative
* plug the micro-USB tip into the Raspberry Pi
* plug RCA male-to-male adapter into the Raspberry Pi and the 3.2” LCD screen
* plug numpad in Raspberry Pi’s USB port
*enclose Raspberry Pi, step-down regulator, and wires (leaving hole for ethernet cable--otherwise plug in ethernet cable first)

Steps to set up Software Environment:
--------------------------------------
* initial raspbian setup
   - no ssh
   - boot into graphical session

* install and configure needed software

        $ sudo apt-get update
        $ sudo apt-get install python-requests python-qrencode

   - download pos files run.py, btcpos.py, lcd.py, bitcoin_accepted.gif to /home/pi (assuming user pi)
   - set run as executable

            $ chmod a+x run.py

   - open run.py and fill in lines identifier, password, and address with your blockchain identifier, password and the forwarding address with which to forward payments. Leave address as ‘’ to simply leave the payments in their respective addresses.

* configure system to remove lxpanel, remove window decorations and autostart program
   - open file /etc/xdg/lxsession/LXDE/autostart and remove the line that begins @ lxpanel
   - open file ~/.config/openbox/lxde-rc.xml and add the following in between the `<applications>` block:

            <application class "*">
                <decor>no</decor>
            </application>

   - make directory for lxde autostarted programs if not already there

            $ mkdir ~/.config/autostart

   - create file ~/.config/autostart/run.desktop with the following:

            [Desktop Entry]
            Type=Application
            Exec=sudo /home/pi/run.py

Security Considerations:
------------------------
* the blockchain.info identifier and password are stored on the device in plaintext -- don't store more money on the account than you can afford to lose
* sending payments to a forwarding address currently defaults with a 0.0005BTC transaction fee
* Don’t use a usb wireless adapter, use an ethernet cable. Using a wireless adapter is just asking for trouble from snoopers (even though all internet communication is supposedly encrypted over SSL)
* Ideally, use an enclosure that doesn’t expose any ports to the pi.
* Double check that forwarding address if you use it.
* Payment is defaulted to be ‘confirmed’ by the device after the unconfirmed transaction is reported. Watch out for double-spends! You’d probably best not to use this for high value purchases.
* All in all, I’d say the device is pretty experimental. It has worked well for me, and doesn’t seem to be glitchy in the slightest, but I would certainly use it at your own risk. ;)

If you like this project, think about tossing a few satoshis this way--> 17yHkgQooxxMdB7iqNbBXDrV8YdMrMes5d
