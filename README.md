# Colosseum_Wireless_Network_Emulator
This project exploits GNU Radio IEEE 802.11 transceiver to instantiate a ad hoc network via Wi-Fi on the Colosseum Wireless Network Emulator. Key tasks aim to connect nodes, make reservations, verify setup, start Wi-Fi nodes, monitor packets, configure scenarios, run scripts, analyze spectrum, and troubleshooting.

In this assignment, we are going to use one of the base Colosseum scenarios (1009), which supports up to 10 nodes in the reservation. The center frequency of this scenario is 1 GHz. This scenario does not add any additional channel characteristics to the RF transmissions of the nodes (besides the contributions of the hardware components of Colosseum). Read the full specifications of this scenario at the following page: https://colosseumneu.freshdesk.com/support/solutions/articles/61000277641-test-scenario-all-paths-0-db-1009.

## Part 1: Connect to Colosseum Connect to the Colosseum wireless network emulator (https://experiments.colosseum.net/).

Following are the steps for the setup:

- Setup Colosseum VPN: https://colosseumneu.freshdesk.com/support/solutions/articles/61000285824-cisco-anyconnect-remote-vpn-access
- Upload your SSH public key on Colosseum: https://colosseumneu.freshdesk.com/support/solutions/articles/61000253402-upload-ssh-public-keys
- Setup your local SSH proxy: https://colosseumneu.freshdesk.com/support/solutions/articles/61000253369-ssh-proxy-setup
- Access Colosseum resources: https://colosseumneu.freshdesk.com/support/solutions/articles/61000253362-accessing-colosseum-resources
  
### Bonus Guide: 
- Quick start guide: https://colosseumneu.freshdesk.com/support/solutions/articles/61000253395-quick-start-guide
- User guide: https://colosseumneu.freshdesk.com/support/solutions/articles/61000253387-colosseum-user-guide

## Part 2: Make a reservation with Wi-Fi nodes on Colosseum

- Make a 2-hour reservation under a suitable name with two colosseum nodes, Standard Radio Nodes (SRNs), with the webinar-interactive-v1 image.
- In the reservation page, find the assigned SRNs/nodes and their hostnames by hovering over nodes.
- At the scheduled time, open two terminals and ssh as root user into the assigned Colosseum SRNs. (PS: the password for the webinar-interactive- v1 container is 'sunflower': "ssh root@<srn-hostname>")

- Now start the scenario: in one of the terminals, run the following command to start a Colosseum Radio-frequency (RF) scenario through the Colosseum CLI API*: "colosseumcli rf start 1009 -c" (The -c option ensures that the scenario automatically restarts after its completion.)
- This will engage the Colosseum RF Channel Emulator and make the necessary connections between the USRPs of the reserved nodes based on the parameters set in the specific RF scenario**.

- You can check if the RF scenario is active and running by executing the following command: "colosseumcli rf info"

## Part 3: Verification of the RF emulator is setup 
In this step, we will verify that Colosseum RF emulator has been setup correctly. Note that you will need the two terminals of the previous step to run the following commands.

- In both terminals, cd to (move to) the /root/utils directory.
- Execute the uhd_tx_tone.sh script in the first terminal: "./uhd_tx_tone.sh"
- Execute the uhd_rx_fft.sh in the second terminal: "./uhd_rx_fft.sh"
- This will send a tone at a set frequency from the first SRN and it will display a spectrum analyzer in the second SRN.
- If the RF emulator is set appropriately as explained in the previous section, the signal generated by the first SRN will propagate through Colosseum RF emulator and reach the second SRN.
  
- Once done, hit Ctrl+C in both terminals to stop the "uhd_tx_tone.sh" and "uhd_rx_fft.sh" example scripts.

## Part 4: Start the Wi-Fi nodes 
In this step, we will use the Wi-Fi nodes reserved in the previous steps. Note that you will need the two terminals of the previous step to run the following commands.

- For each SRN, navigate to the directory /root/interactive_scripts and execute the tap_setup.sh script to setup a tap interface for the SRN (see Traffic Generation for more information on routing traffic in Colosseum): "/root/interactive_scripts/tap_setup.sh"
- For each SRN, execute the route_setup.sh script (located in the /root/interactive_scripts directory) to setup the routing tables: /root/interactive_scripts/route_setup.sh
- NOTE: In each of the SRN, you need to setup the route to the other SRN. The SRN IDs are created by adding 100 to the SRN number assigned to your reservation. As an example, SRN- 015 will have ID 115, hence the IP address of the tr0 interface of this node will be 192.168.115.1. In this example, the above command becomes: "/root/interactive_scripts/route_setup.sh 115"
- In each SRN, execute the modem_start.sh script to start the Wi-Fi modem: "/root/interactive_scripts/modem_start.sh"
- NOTE: we need to leave these two processes running to be able to communicate between the Wi-Fi nodes, and perform the following steps.

- Now, open two new terminals and ssh into the same SRNs as before (this is to keep the other terminals with the modem running).
- From each SRN, in the newly opened terminals, ping the tr0 interface of the other SRN: ping 192.168..1 This transmits ICMP packets over the RF emulator.
- If the ping is successful, it means that you have configured your SRNs in the correct way and you have an emulated channel between them.
- Also observe new Wi-Fi packets being generated and sent in the previous two terminals.
- Once done, hit Ctrl+C to stop the ping in both SRNs.

## Part 5: Start Colosseum Traffic Generator (TGEN) 
In this step, we will use the Wi-Fi nodes reserved in the previous steps. Note that you will need the two terminals of the previous step to run the following commands.

- In one of the terminals, execute the following command to start a traffic scenario through the Colosseum traffic emulator: "colosseumcli tg start 10090"
- This will engage the Colosseum Traffic Generator (TGEN) and start packet flows between Colosseum and the SRNs of your reservation based on the parameters specified in the traffic scenario (see https://colosseumneu.freshdesk.com/support/solutions/articles/61000277641-test- scenario-all-paths-0-db-1009).
- You can check if the traffic scenario is active (and running) by executing the following command: "colosseumcli tg info"
- Now you can monitor the packet flow on the tr0 interface (i.e., the interface in which the SRNs receive/forward packets from from/to TGEN) of each SRN by running the following command on each SRN: "tcpdump -i tr0"
- Note that it takes a few minutes (~5 mins) for TGEN to initialize the traffic scenario and start transmitting packets to the SRNs.
- Once done, hit Ctrl+C in all four terminals to stop the Wi-Fi applications and the tcpdump.

## Part 6: Clean up!
This concludes Colosseum Wi-Fi project.

- Once done with experimenting, it is good practice to stop the traffic and RF and TGEN scenarios by running the following commands from in one of the previous terminals: "colosseumcli rf stop" & "colosseumcli tg stop"
- Close all the previous terminals.
- Log into the Colosseum website and terminate your reservation by clicking the red X button next to your reservation.

## References: 
-  Colosseum CLI API: https://colosseumneu.freshdesk.com/support/solutions/articles/61000253397-colosseum-cli
-  Test Scenario All Paths 0 dB (1009): https://colosseumneu.freshdesk.com/support/solutions/articles/61000277641-test-scenario-all-paths-0-db-1009
