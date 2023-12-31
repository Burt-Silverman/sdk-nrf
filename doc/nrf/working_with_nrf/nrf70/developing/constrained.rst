.. _nRF7002dk_nRF5340_constrained_host:

Operating with a resource constrained host
##########################################

.. contents::
   :local:
   :depth: 2

This guide provides recommendations and guidelines for using the nRF7002 as a companion chip on resource-constrained hosts such as the nRF5340 SoC.

Zephyr OS factors
*****************
The following sections explain the factors that are applicable when using the Zephyr OS on the nRF5340 SoC.

CPU frequency
=============

The nRF5340 host has two operating frequencies, 64 MHz and 128 MHz, default frequency is 64 MHz.
For low power applications, it is recommended to use 64 MHz as the CPU frequency but the nRF7002 Wi-Fi performance might be impacted.
For high performance applications, it is recommended to use 128 MHz as the CPU frequency.

Networking stack
================

The nRF7002 driver uses the Zephyr networking stack for Wi-Fi protocol implementation.
You can configure the networking stack to use different networking buffers and queue depths.
The nRF7002 driver can be configured to use different numbers of TX buffers and RX buffers based on the use case.
A separate configuration option is provided to configure the number of packets, and the number of buffers used by each packet can also be configured.

The following table explains the configuration options:

+------------------------------------------+-----------------------------+--------------------------------------------------------------------------------------------------------------------------+
|Configuration Option                      | Values                      | Description                                                                                                              |
+==========================================+=============================+==========================================================================================================================+
|:kconfig:option:`CONFIG_NET_PKT_TX_COUNT` | ``1`` - Unlimited           | Number of TX packets. This is the TX queue depth, higher depth saturates the Wi-Fi link but consumes more memory,        |
|                                          | (based on available memory) | lower depth reduces memory usage but does not saturate the Wi-Fi link leading to poor performance.                       |
+------------------------------------------+-----------------------------+--------------------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_NET_PKT_RX_COUNT`| ``1`` - Unlimited           | Number of RX packets. This is the RX queue depth, higher depth can keep up with the RX traffic but consumes more memory, |
|                                          | (based on available memory) | lower depth reduces memory usage but does not keep up with the RX traffic leading to packet drops.                       |
+------------------------------------------+-----------------------------+--------------------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_NET_BUF_TX_COUNT`| ``1`` - Unlimited           | Number of TX buffers. Typically for Wi-Fi, each packet has two buffers,                                                  |
|                                          | (based on available memory) | so this has to be twice the number of packets.                                                                           |
+------------------------------------------+-----------------------------+--------------------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_NET_BUF_RX_COUNT`| ``1`` - Unlimited           | Number of RX buffers. Typically for Wi-Fi, each packet has one buffer,                                                   |
|                                          | (based on available memory) | so this has to be equal to the number of packets.                                                                        |
+------------------------------------------+-----------------------------+--------------------------------------------------------------------------------------------------------------------------+

nRF700X driver performance and memory fine-tuning controls
**********************************************************

The nRF700x driver provides the following software configurations to fine-tune memory and performance based on the use case:

+------------------------------------------+------------------------------+-----------------------------------------------------------------------------------+----------------------------------------+---------------------------------------------------------------------------------------------------------------+
|Configuration Option                      | Values                       | Description                                                                       | Impact                                 | Purpose                                                                                                       |
+==========================================+==============================+===================================================================================+========================================+===============================================================================================================+
| :kconfig:option:`CONFIG_WPA_SUPP`        | ``y`` or ``n``               | Enable or disable Wi-Fi Protected Access (WPA) supplicant                         | Memory savings                         | This specifies the inclusion of the WPA supplicant module.                                                    |
|                                          |                              |                                                                                   |                                        | Disabling this flag restricts the nRF700x driver's functionality to STA scan only.                            |
+------------------------------------------+------------------------------+-----------------------------------------------------------------------------------+----------------------------------------+---------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_NRF700X_AP_MODE` | ``y`` or ``n``               | Enable or disable Access Point (AP) mode                                          | Memory savings                         | This specifies the inclusion of the AP mode module.                                                           |
|                                          |                              |                                                                                   |                                        | Disabling this flag restricts the nRF700x driver's functionality to :term:`Station mode (STA)` only.          |
+------------------------------------------+------------------------------+-----------------------------------------------------------------------------------+----------------------------------------+---------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_NRF700X_P2P_MODE`| ``y`` or ``n``               | Enable or disable Wi-Fi direct mode                                               | Memory Savings                         | This specifies the inclusion of the P2P mode module.                                                          |
|                                          |                              |                                                                                   |                                        | Disabling this flag restricts the nRF700x driver's functionality to STA or AP mode only.                      |
+------------------------------------------+------------------------------+-----------------------------------------------------------------------------------+----------------------------------------+---------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_MAX_TX_TOKENS`   | ``5``, ``10``, ``11``, ``12``| Maximum number of TX tokens.                                                      | Performance tuning and Memory savings  | This specifies the maximum number of TX tokens that can be used in the token bucket algorithm.                |
|                                          |                              | These are distributed across all WMM access categories (including a pool for all).|                                        | More tokens imply more concurrent transmit opportunities for RPU but can lead to poor aggregation performance |
|                                          |                              |                                                                                   |                                        | if the pipeline is not saturated. But to saturate the pipeline, a greater number of networking stack buffers, |
|                                          |                              |                                                                                   |                                        | or queue depth, is required.                                                                                  |
+------------------------------------------+------------------------------+-----------------------------------------------------------------------------------+----------------------------------------+---------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_RX_NUM_BUFS`     | ``1`` to ``16``              | Number of RX buffers                                                              | Memory savings                         | This specifies the number of RX buffers that can be used by the nRF700x driver.                               |
|                                          |                              |                                                                                   |                                        | The number of buffers must be enough to keep up with the RX traffic, otherwise packets might be dropped.      |
+------------------------------------------+------------------------------+-----------------------------------------------------------------------------------+----------------------------------------+---------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_TX_MAX_DATA_SIZE`| ``64`` to ``1600``           | Maximum TX data size                                                              | Memory savings                         | This specifies the maximum size of Wi-Fi protocol frames that can be transmitted.                             |
|                                          |                              |                                                                                   |                                        | Large frame sizes imply more memory usage but can efficiently utilize the bandwidth.                          |
|                                          |                              |                                                                                   |                                        | If the application does not need to send large frames, then this can be reduced to save memory.               |
+------------------------------------------+------------------------------+-----------------------------------------------------------------------------------+----------------------------------------+---------------------------------------------------------------------------------------------------------------+
| :kconfig:option:`CONFIG_RX_MAX_DATA_SIZE`| ``64`` to ``1600``           | Maximum RX data size                                                              | Memory savings                         | This controls the maximum size of the frames that can be received by the Wi-Fi protocol.                      |
|                                          |                              |                                                                                   |                                        | Large frame sizes imply more memory usage but can efficiently utilize the bandwidth.                          |
|                                          |                              |                                                                                   |                                        | If the application does not need to receive large frames, then this can be reduced to save memory.            |
+------------------------------------------+------------------------------+-----------------------------------------------------------------------------------+----------------------------------------+---------------------------------------------------------------------------------------------------------------+

The configuration options must be used in conjunction with the Zephyr networking stack configuration options to achieve the desired performance and memory usage.
These options form a staged pipeline all the way to the nRF7002 chip, any change in one stage of the pipeline will impact the performance and memory usage of the next stage.
For example, solving bottleneck in one stage of the pipeline might lead to a bottleneck in the next stage.

Usage profiles
**************

The nRF700x driver can be used in the following profiles (not an exhaustive list):

.. list-table::
   :header-rows: 1

   * - Features
     - Profile
     - Configuration Options
     - Use cases
   * - STA scan only
     - Scan only
     - ``CONFIG_WPA_SUPP=n``
       ``CONFIG_NRF700X_AP_MODE=n``
       ``CONFIG_NRF700X_P2P_MODE=n``
     - Location services
   * - :abbr:`STA (Station)` mode
     - IoT devices
     - ``CONFIG_WPA_SUPP=y``
       ``CONFIG_NRF700X_AP_MODE=n``
       ``CONFIG_NRF700X_P2P_MODE=n``
     - IoT devices
   * - :abbr:`STA (Station)` mode
     - Memory optimized :abbr:`STA (Station)` mode
     - ``CONFIG_MAX_TX_TOKENS=5``
       ``CONFIG_RX_NUM_BUFS=4``
       ``CONFIG_TX_MAX_DATA_SIZE=512``
       ``CONFIG_RX_MAX_DATA_SIZE=512``
     - Sensors with low data requirements
   * - :abbr:`STA (Station)` mode
     - High performance :abbr:`STA (Station)` mode
     - ``CONFIG_MAX_TX_TOKENS=12``
       ``CONFIG_RX_NUM_BUFS=64``
       ``CONFIG_TX_MAX_DATA_SIZE=1600``
       ``CONFIG_RX_MAX_DATA_SIZE=1600``
     - High data rate IoT devices
   * - :abbr:`STA (Station)` mode
     - TX prioritized :abbr:`STA (Station)` mode
     - ``CONFIG_MAX_TX_TOKENS=12``
       ``CONFIG_RX_NUM_BUFS=4``
       ``CONFIG_TX_MAX_DATA_SIZE=1600``
       ``CONFIG_RX_MAX_DATA_SIZE=512``
     - Sensors with high data rate
   * - :abbr:`STA (Station)` mode
     - RX prioritized :abbr:`STA (Station)` mode
     - ``CONFIG_MAX_TX_TOKENS=5``
       ``CONFIG_RX_NUM_BUFS=64``
       ``CONFIG_TX_MAX_DATA_SIZE=512``
       ``CONFIG_RX_MAX_DATA_SIZE=1600``
     - Display devices streaming data
