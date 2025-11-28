# Android Kotlin VPN Firewall & Traffic Logger

This repository contains an Android application that demonstrates how to use Android's `VpnService` to monitor and log network traffic for user-selected applications. It acts as a local firewall, routing traffic from specific apps through a VPN service where packets are parsed, logged, and then forwarded to their original destination.

The application is built using a modern Android development stack, including Kotlin, Jetpack Compose, Coroutines, Flow, Koin for dependency injection, Room for local database storage, and DataStore for preferences.

## Features

-   **VPN-Based Traffic Interception**: Utilizes `VpnService` to create a local VPN and route network traffic from selected apps.
-   **Selective App Monitoring**: Users can choose which installed applications they want to monitor.
-   **Real-time Traffic Logging**: Captures and displays network connection details, including application name, protocol (TCP/UDP), source/destination IP addresses, and ports.
-   **Native Packet Parsing**: Integrates with a native C++ library via JNI (`NativeBridge.kt`) for high-performance parsing of IP packets.
-   **Modern UI**: A clean, responsive UI built entirely with Jetpack Compose.
-   **Persistent State**: Uses Room to store traffic logs and Proto DataStore to persist the list of monitored apps.
-   **Asynchronous Operations**: Leverages Kotlin Coroutines and Flow for managing background tasks like network operations and database access.

## How It Works

The core of the application is the `FirewallVpnService`. When activated, it establishes a local VPN on the device.

1.  **Configuration**: The `VpnService` is configured to only handle traffic for the applications the user has selected for monitoring. Traffic from all other apps bypasses the VPN, as they are added to the service's `disallowed` list.
2.  **Packet Interception**: IP packets from the monitored apps are sent to the VPN's virtual network interface. `FirewallVpnService` reads these packets as a raw byte stream from the interface's file descriptor.
3.  **Native Parsing**: Each raw packet is passed to a native C++ library through the `NativeBridge`. The native code parses the IPv4 header and the transport layer header (TCP/UDP) to extract details like protocol, source/destination IPs, and ports.
4.  **UID Resolution**: The service uses `UidResolver` to determine which application (by its UID) owns the connection, which is then mapped to the application's name.
5.  **Logging**: The parsed connection information is encapsulated in a `BlockLogEntity` and saved to a local Room database. The UI displays these logs in real-time.
6.  **Packet Forwarding**: After logging, the application forwards the packets to their original destination.
    -   **UDP**: UDP packets' payloads are extracted and forwarded using a protected `DatagramSocket`. The response is received and written back to the VPN interface.
    -   **TCP**: For TCP, a new `Socket` is created for each connection and protected from the VPN. The payload is written to this socket, and a separate coroutine listens for server responses to write back to the VPN interface.
7.  **UI & State Management**: The UI is built with Jetpack Compose. `MainViewModel` handles the application's state, fetches logs from the Room database, manages the list of monitored apps from DataStore, and interacts with the `VpnService`.

## Core Components

-   `FirewallVpnService.kt`: Manages the VPN lifecycle, reads packets from the virtual interface, orchestrates parsing and logging, and forwards traffic.
-   `NativeBridge.kt`: A Kotlin object that defines the JNI interface to the native C++ packet-parsing functions (`libfirewallapp.so`).
-   `MainViewModel.kt`: A Koin-injected ViewModel that holds UI state, exposes traffic logs and the list of monitored apps as `StateFlow`, and handles user interactions.
-   `data/repository/BlockedAppRepositoryImpl.kt`: Manages the list of monitored app package names using Jetpack DataStore.
-   `data/local/`: Contains the Room database (`BlockLogDatabase`), DAO (`BlockLogDao`), and entity (`BlockLogEntity`) for persisting traffic logs.
-   `ui/screen/`: Contains the main Jetpack Compose screens for displaying traffic logs (`Main_Screen.kt`), managing monitored apps (`Blocked_Apps.kt`), and selecting apps (`Select_Apps.kt`).
-   `ui/bottomBar/`: Implements the main navigation structure, including the `TopAppBar` with the VPN toggle switch and the `BottomNavigationBar`.

## How to Use

1.  Launch the application.
2.  Tap the floating action button (+) to navigate to the "Select apps to block" screen.
3.  A list of installed applications will be displayed. Tap the block icon next to any app you wish to monitor. The icon will turn red for apps that are already on the monitoring list.
4.  Navigate back to the main screen. The apps you selected will appear under the "Blocked Apps" tab.
5.  In the top app bar, toggle the switch to the "on" position. This will prompt you for VPN permissions if it's the first time.
6.  Once the VPN is active, any network traffic from the monitored apps will be logged.
7.  Switch to the "Traffic Logs" tab to view a real-time list of captured network connections.
8.  To stop the service, simply toggle the switch in the top app bar to the "off" position.
