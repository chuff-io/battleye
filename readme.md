<p align="center">
  <a href="https://www.npmjs.com/package/battleye"><img src="https://img.shields.io/npm/dm/battleye.svg" alt="Downloads"></a>
  <a href="https://www.npmjs.com/package/battleye"><img src="https://img.shields.io/npm/v/battleye.svg" alt="Version"></a>
  <a href="https://www.npmjs.com/package/battleye"><img src="https://img.shields.io/npm/l/battleye.svg" alt="License"></a>
</p>

# battleye

> Battleye rcon client built in nodejs.

## Example usage:
```js
import * as readline from 'readline'
import { readCfg, Socket } from 'battleye'

readCfg(process.cwd())
  .then(cfg => {
    console.log(cfg)

    if (!cfg.rconpassword || !cfg.rconip || !cfg.rconport) {
      throw new Error('Invalid BEServer.cfg')
    }

    // create socket
    const socket = new Socket({
      port: 2310,     // listen port
      ip: '0.0.0.0',  // listen ip
    })

    // create connection
    const connection = socket.connection({
      name: 'my-server',                // server name
      password: cfg.rconpassword,       // rcon password
      ip: cfg.rconip,                   // rcon ip
      port: cfg.rconport                // rcon port
    }, {
      reconnect: true,              // reconnect on timeout
      reconnectTimeout: 500,        // how long (in ms) to try reconnect
      keepAlive: true,              // send keepAlive packet
      keepAliveInterval: 15000,     // keepAlive packet interval (in ms)
      timeout: true,                // timeout packets
      timeoutInterval: 1000,        // interval to check packets (in ms)
      serverTimeout: 30000,         // timeout server connection (in ms)
      packetTimeout: 1000,          // timeout packet check interval (in ms)
      packetTimeoutThresholded: 5,  // packets to resend
    })

    // create readline for command input
    const rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    })

    socket.on('listening', (socket) => {
      const addr = socket.address()
      console.log(`Socket listening on ${typeof addr === 'string' ? addr : `${addr.address}:${addr.port}`}`)
    })

    socket.on('received', (resolved, packet, buffer, connection, info) => {
      console.log(`received: ${connection.ip}:${connection.port} => packet:`, packet)
    })

    socket.on('sent', (packet, buffer, bytes, connection) => {
      console.log(`sent: ${connection.ip}:${connection.port} => packet:`, packet)
    })

    socket.on('error', (err) => { console.error(`SOCKET ERROR:`, err) })

    connection.on('message', (message, packet) => {
      console.log(`message: ${connection.ip}:${connection.port} => message: ${message}`)
    })

    connection.on('command', (data, resolved, packet) => {
      console.log(`command: ${connection.ip}:${connection.port} => packet:`, packet)
    })

    connection.on('disconnected', (reason) => {
      console.warn(`disconnected from ${connection.ip}:${connection.port},`, reason)
    })

    connection.on('connected', () => {
      console.error(`connected to ${connection.ip}:${connection.port}`)
    })

    connection.on('debug', console.log)

    connection.on('error', (err) => {
      console.error(`CONNECTION ERROR:`, err)
    })

    rl.on('line', input => {
      connection
        .command(input)
        .then(response => {
          console.log(`response: ${connection.ip}:${connection.port} => ${response.command}\n${response.data}`)
        })
        .catch(console.error)

      console.log(`send: ${connection.ip}:${connection.port} => ${input}`)
    })
  })
  .catch(err => { console.error(`Error reading config:`, err) })
```
