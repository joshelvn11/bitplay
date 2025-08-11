# BitPlay: Torrent Streaming Web App

BitPlay is a web application built with Go that allows you to stream video content directly from torrents in your browser. It features a clean web UI, support for SOCKS5 proxies, and integration with Prowlarr and Jackett for seamless torrent searching.

![Bitplay Home](screenshots/bitplay_home.png)

## Features

*   **Direct Torrent Streaming:** Stream video files from magnet links or torrent files directly without needing to download them completely first.
*   **Web-Based UI:** Access and control BitPlay through a user-friendly web interface.
*   **Proxy Support:** Configure a SOCKS5 proxy for all torrent-related traffic (fetching metadata, peer connections). (Note: HTTP proxies are not currently supported).
*   **Prowlarr Integration:** Connect to your Prowlarr instance to search across your configured indexers directly within BitPlay.
*   **Jackett Integration:** Connect to your Jackett instance as an alternative search provider.
*   **On-the-fly Subtitle Conversion:** Converts SRT subtitles to VTT format for browser compatibility.
*   **Session Management:** Handles multiple torrent sessions and cleans up inactive ones.

## Getting Started

You can run BitPlay either directly using Go or via Docker Compose.

### Prerequisites

*   **Go:** Requires Go 1.18 or later (if running locally).
*   **Docker & Docker Compose:** Required if running with Docker.

### Running Locally with Go

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/aculix/bitplay.git
    cd bitplay
    ```
2.  **Download dependencies:**
    ```bash
    go mod download
    ```
3.  **Run the application:**
    ```bash
    go run main.go
    ```
    By default, the server will start on `http://localhost:3347`.

### Running with Docker Compose

This is the recommended method for deployment.

1.  **Create a `docker-compose.yml` file:**
    ```yaml
    services:
      bitplay:
        image: ghcr.io/aculix/bitplay:main
        container_name: bitplay
        ports:
          - 3347:3347 # Expose the web UI port
        volumes:
          # Mount the config directory for persistent settings (Optional)
          - ./config:/app/config 
        restart: unless-stopped
    ```
    *   **Optional Persistence:** By default, settings (Proxy, Prowlarr/Jackett) are stored inside the container and will be lost if the container is restarted. To make settings persistent across restarts, you can mount a local directory from your host to `/app/config` inside the container using the `volumes` option above. 
    *   If you choose to mount the directory for persistence, you **must** create the directory on your host machine **before** starting the container for the first time: `mkdir -p ./config`. 
    *   If you don't mount this volume, the application will still function correctly, but you will need to re-configure your settings after each container restart. Torrent data itself is always ephemeral.

2.  **Start the container:**
    ```bash
    docker-compose up -d
    ```
3.  **Access the application:** Open your browser to `http://<your-server-ip>:3347`.

### Running with Docker Run

Alternatively, you can run the container directly using `docker run`:

1.  **(Optional) Create the config directory for persistence:** If you want your settings (Proxy, Prowlarr/Jackett) to persist across container restarts, create the configuration directory on your host first:
    ```bash
    mkdir -p ./config
    ```
    If you skip this step and the volume mount below, the application will still work, but settings will be lost on restart.

2.  **Run the container:**
    ```bash
    docker run -d \
      --name bitplay \
      -p 3347:3347 \
      # Add the volume mount below ONLY if you want persistent settings (and created ./config above)
      -v $(pwd)/config:/app/config \
      --restart unless-stopped \
      ghcr.io/aculix/bitplay:main
    ```
    *   `-d`: Run in detached mode (background).
    *   `--name bitplay`: Assign a name to the container.
    *   `-p 3347:3347`: Map port 3347 on the host to port 3347 in the container.
    *   `-v $(pwd)/config:/app/config`: (Optional) Mount the local `./config` directory for persistent settings.
    *   `--restart unless-stopped`: Configure the container to restart automatically unless manually stopped.
    *   `ghcr.io/aculix/bitplay:main`: The Docker image to use.

3.  **Access the application:** Open your browser to `http://<your-server-ip>:3347`.

## Configuration

BitPlay is configured primarily through its web interface after starting the application.

1.  **Access the Web UI:** Go to `http://localhost:3347` (or your server's address).
2.  **Navigate to Settings:** Find the settings or configuration section within the UI.
3.  **Configure:**
    *   **Proxy:** Enable/disable proxy support and provide the full SOCKS5 proxy URL (e.g., `socks5://user:pass@host:port`). Test the connection using the provided button.
    *   **Prowlarr:** Enable/disable Prowlarr, provide the Prowlarr Host URL (e.g., `http://prowlarr:9696`), and your Prowlarr API Key. Test the connection.
    *   **Jackett:** Enable/disable Jackett, provide the Jackett Host URL (e.g., `http://jackett:9117`), and your Jackett API Key. Test the connection.

Settings are saved automatically to `/app/config/settings.json` inside the Docker container, which maps to `./config/settings.json` on the host via the mounted volume in the example Docker Compose setup above.

## Usage

1.  **Configure Settings:** Set up your proxy and search providers (Prowlarr/Jackett) as described above.
2.  **Search:** Use the search bar to query Prowlarr or Jackett for torrents.
3.  **Add Torrent:** Paste a magnet link directly or click a search result to add the torrent to BitPlay.

    > BitPlay also supports opening from another app/program by providing torrent or magnet link as query parameter `torrent`. The link needs to be URL-encoded! 
    >
    > Ex: `http://localhost:3347?torrent=magnet%3A%3Fxt%3Durn%3Abtih%3A08ada5a7a6183aae1e09d831df6748d566095a10%26dn%3DSintel%26tr%3Dudp%253A%252F%252Fexplodie.org%253A6969%26tr%3Dudp%253A%252F%252Ftracker.coppersurfer.tk%253A6969%26tr%3Dudp%253A%252F%252Ftracker.empire-js.us%253A1337%26tr%3Dudp%253A%252F%252Ftracker.leechers-paradise.org%253A6969%26tr%3Dudp%253A%252F%252Ftracker.opentrackr.org%253A1337%26tr%3Dwss%253A%252F%252Ftracker.btorrent.xyz%26tr%3Dwss%253A%252F%252Ftracker.fastcast.nz%26tr%3Dwss%253A%252F%252Ftracker.openwebtorrent.com%26ws%3Dhttps%253A%252F%252Fwebtorrent.io%252Ftorrents%252F%26xs%3Dhttps%253A%252F%252Fwebtorrent.io%252Ftorrents%252Fsintel.torrent`

4.  **Stream:** Once the torrent info is loaded, select the video file you want to watch. BitPlay will start downloading and streaming it directly in the built-in player.

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues. (You can add more details here if you have specific contribution guidelines).

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
