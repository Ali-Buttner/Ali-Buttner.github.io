<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Arduino Web Flasher</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/avrgirl-arduino/4.3.0/avrgirl-arduino.global.min.js"></script>
    <style>
        :root {
            --primary: #00979D;
            --primary-dark: #007076;
            --secondary: #e47128;
            --light: #f5f5f5;
            --dark: #333;
            --success: #28a745;
            --error: #dc3545;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            color: var(--dark);
            background-color: var(--light);
            margin: 0;
            padding: 0;
        }
        
        .container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        header {
            background-color: var(--primary);
            color: white;
            padding: 1rem;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        
        h1 {
            margin: 0;
            font-size: 2rem;
        }
        
        .card {
            background: white;
            border-radius: 8px;
            padding: 20px;
            margin: 20px 0;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        .button {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
            transition: background-color 0.3s;
            width: 100%;
            margin-bottom: 10px;
        }
        
        .button:hover {
            background-color: var(--primary-dark);
        }
        
        .button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
        
        .button-secondary {
            background-color: var(--secondary);
        }
        
        #log {
            background-color: #f8f9fa;
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 10px;
            height: 200px;
            overflow-y: auto;
            font-family: monospace;
            white-space: pre-wrap;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        select {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            background-color: white;
        }
        
        .alert {
            padding: 10px;
            border-radius: 4px;
            margin-bottom: 10px;
        }
        
        .alert-error {
            background-color: #f8d7da;
            color: var(--error);
            border: 1px solid #f5c6cb;
        }
        
        .alert-success {
            background-color: #d4edda;
            color: var(--success);
            border: 1px solid #c3e6cb;
        }
        
        .status-indicator {
            display: inline-block;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            margin-right: 5px;
        }
        
        .status-connected {
            background-color: var(--success);
        }
        
        .status-disconnected {
            background-color: var(--error);
        }
        
        footer {
            text-align: center;
            padding: 1rem;
            background-color: var(--primary);
            color: white;
            margin-top: 20px;
        }
        
        .hidden {
            display: none;
        }
        
        .steps {
            display: flex;
            justify-content: space-between;
            margin-bottom: 20px;
        }
        
        .step {
            flex: 1;
            text-align: center;
            padding: 15px;
            border-bottom: 3px solid #ddd;
        }
        
        .step.active {
            border-bottom-color: var(--primary);
            font-weight: bold;
        }
        
        .step.completed {
            border-bottom-color: var(--success);
        }
        
        .firmware-info {
            background-color: #f0f0f0;
            border-left: 4px solid var(--primary);
            padding: 10px 15px;
            margin: 15px 0;
        }
        
        @media (max-width: 600px) {
            .container {
                padding: 10px;
            }
            
            .steps {
                flex-direction: column;
            }
            
            .step {
                margin-bottom: 10px;
            }
        }
    </style>
</head>
<body>
    <header>
        <h1>Arduino Web Flasher</h1>
        <p>Flash your Arduino in two simple steps</p>
    </header>
    
    <div class="container">
        <div class="steps">
            <div id="step1" class="step active">1. Connect Device</div>
            <div id="step2" class="step">2. Flash Firmware</div>
        </div>
        
        <div class="card">
            <h2>Device Connection</h2>
            <p>
                <span id="connectionStatus" class="status-indicator status-disconnected"></span>
                <span id="statusText">No device connected</span>
            </p>
            <button id="connectButton" class="button">Connect Arduino</button>
            <button id="disconnectButton" class="button" disabled>Disconnect</button>
        </div>
        
        <div id="flashSection" class="card hidden">
            <h2>Ready to Flash</h2>
            
            <div class="firmware-info">
                <h3>Firmware Details</h3>
                <p><strong>Name:</strong> <span id="firmwareName">Blink Example</span></p>
                <p><strong>Description:</strong> <span id="firmwareDescription">Basic LED blinking program that flashes the built-in LED on pin 13</span></p>
                <p><strong>Version:</strong> <span id="firmwareVersion">1.0.0</span></p>
            </div>
            
            <div class="form-group">
                <label for="boardType">Select Your Board:</label>
                <select id="boardType">
                    <option value="uno">Arduino Uno</option>
                    <option value="mega">Arduino Mega</option>
                    <option value="nano">Arduino Nano</option>
                    <option value="leonardo">Arduino Leonardo</option>
                    <option value="micro">Arduino Micro</option>
                </select>
            </div>
            
            <button id="flashButton" class="button button-secondary">Flash Arduino</button>
        </div>
        
        <div class="card">
            <h2>Console Output</h2>
            <div id="log"></div>
        </div>
    </div>
    
    <footer>
        <p>Created with ❤️ for the Arduino community | <a href="https://github.com/yourusername/arduino-web-flasher" style="color: white;">View on GitHub</a></p>
    </footer>
    
    <script>
        // DOM Elements
        const connectButton = document.getElementById('connectButton');
        const disconnectButton = document.getElementById('disconnectButton');
        const flashButton = document.getElementById('flashButton');
        const statusText = document.getElementById('statusText');
        const connectionStatus = document.getElementById('connectionStatus');
        const flashSection = document.getElementById('flashSection');
        const boardTypeSelect = document.getElementById('boardType');
        const logElement = document.getElementById('log');
        const step1 = document.getElementById('step1');
        const step2 = document.getElementById('step2');
        
        // Global variables
        let port = null;
        let reader = null;
        let inputDone = null;
        let outputDone = null;
        let inputStream = null;
        let outputStream = null;
        
        // Pre-loaded hex file for the Blink sketch (replace with your own)
        const preloadedHex = `:100000000C945C000C946E000C946E000C946E00CA
:100010000C946E000C946E000C946E000C946E00A8
:100020000C946E000C946E000C946E000C946E0098
:100030000C946E000C946E000C946E000C946E0088
:100040000C9413010C946E000C946E000C946E00D2
:100050000C946E000C946E000C946E000C946E0068
:100060000C946E000C946E00000000002400270029
:100070002A0000000000250028002B0004040404CE
:100080000404040402020202020203030303030342
:10009000010204081020408001020408102001021F
:1000A00004081020000000080002010000030407FB
:1000B000000000000000000011241FBECFEFD8E0B8
:1000C000DEBFCDBF21E0A0E0B1E001C01D92A930AC
:1000D000B207E1F70E945D010C94CC010C94000078
:1000E00090E0FC01E859FF4F2491FC01EC55FF4FD2
:1000F0003491FC01E057FF4FE491EE23C9F0222307
:1001000039F0233001F1A8F4213019F1223029F11E
:10011000F0E0EE0FFF1FE458FF4FA591B4918FB7A9
:10012000F894EC91611126C030953E233C938FBF2B
:1001300008952730A9F02830C9F0243049F780917D
:1001400080008F7D03C0809180008F77809380002C
:10015000DFCF84B58F7784BDDBCF84B58F7DFBCFB8
:100160008091B0008F778093B000D2CF8091B00056
:100170008F7DF9CF3E2BDACF3FB7F8948091050100
:1001800090910601A0910701B091080126B5A89BA6
:1001900005C02F3F19F00196A11DB11D3FBFBA2F19
:1001A000A92F982F8827BC01CD01620F711D811DD9
:1001B000911D42E0660F771F881F991F4A95D1F7E1
:1001C0000895CF92DF92EF92FF92CF93DF936B01BF
:1001D0007C010E94B800EB01C114D104E104F104AE
:1001E00089F00E9488000E94B8006C1B7D0B683EA0
:1001F000734080F381E0C81AD108E108F108C8517F
:10020000DC4FEACFDF91CF91FF90EF90DF90CF90CB
:10021000089543E050E062E171E083E091E00C945C
:100220003E01CF93DF9390E0FC01EC55FF4F249103
:100230008057FF4F8491882349F190E0880F991FF1
:10024000FC01EA57FF4FA591B491FC01E458FF4F41
:10025000C591D4916623A1F42FB7F8948C91932FBC
:10026000909589238C93888189230BC0623061F491
:100270002FB7F8948C91932F909589238C93888198
:10028000832B88832FBF06C09FB7F8948C91832BD6
:100290008C939FBFDF91CF9108951F920F920FB623
:1002A0000F9211242F933F938F939F93AF93BF93E9
:1002B0008091010190910201A0910301B0910401DC
:1002C00030910001232F2A570196A11DB11D209333
:1002D00000018093010190930201A0930301B09359
:1002E00004018091050190910601A0910701B091A4
:1002F00008010196A11DB11D809305019093060138
:10030000A0930701B0930801BF91AF919F918F9185
:100310003F912F910F900FBE0F901F901895789408
:1003200084B5826084BD84B5816084BD85B582608F
:1003300085BD85B5816085BD80916E008160809365
:100340006E00109281008091810082608093810052
:1003500080918100816080938100809180008160F3
:10036000809380008091B10084608093B1008091D8
:10037000B00081608093B00080917A0084608093CA
:100380007A0080917A00826080937A0080917A001C
:10039000816080937A0080917A00806880937A009D
:1003A0001092C10082E00E94100188EF93E00E94EA
:1003B000C00160E082E00E94070160E088EF93E00D
:1003C0000E94C80162E088EF93E00E94C801C0E0BB
:1003D000D0E0D0937F01C0937E0183E091E00E94F8
:1003E000340100937E0110937F0183E091E00E94AD
:1003F0003401809168008E7F8093680080916B0054
:10040000846080936B0080916D00846080936D001F
:100410008EE20E94100170E060E088EF93E00E94C5
:10042000C8018FE30E94100170E060E08CEF93E00E
:100430000E94C8010E948F00882389F020E030E045
:1004400048EC52E483E091E00E9440010E94B80083
:100450002B013C0180E3C82ED12CE12CF12CE114D2
:10046000F10409F46BC08091000181112EC0809170
:1004700001019091020181309105D9F080916B0088
:10048000846080936B0080916D00846080936D0075
:10049000809100018F5F809300016CEF70E080E023
:1004A00090E00E94BD00C62FD0E0CF5DDE4F888138
:1004B0000E94100147C080916D008B7F80936D00CD
:1004C00080916B008B7F80936B00D501C401820F4F
:1004D000931FA41FB51F0896A11DB11DC82FD0E050
:1004E000CF5DDE4F88810E9410012FEFC216D10485
:1004F000E104F10484F461E082E00E94070160E058
:1005000082E00E9407012FEFC216D104E104F1043D
:1005100009F075CF61E078CF0E948F00882309F05F
:1005200072CF62E082E00E940701C11479E0D7061C
:10053000E104F10409F064CF61E082E00E940701F0
:1005400064CF52E0C51AD108E108F108C114D1042A
:10055000E104F10409F480CFCECFF894FFCF00006A
:00000001FF`;
        
        // Log helper function
        function log(message, type = 'info') {
            const timestamp = new Date().toLocaleTimeString();
            const entry = document.createElement('div');
            entry.textContent = `[${timestamp}] ${message}`;
            
            if (type === 'error') {
                entry.style.color = 'var(--error)';
            } else if (type === 'success') {
                entry.style.color = 'var(--success)';
            }
            
            logElement.appendChild(entry);
            logElement.scrollTop = logElement.scrollHeight;
        }
        
        // Check if WebUSB is supported
        if (!navigator.usb) {
            log('WebUSB is not supported in this browser. Please use Chrome or Edge.', 'error');
            connectButton.disabled = true;
        }
        
        // Connect to Arduino
        connectButton.addEventListener('click', async () => {
            try {
                log('Requesting device...');
                
                // Arduino USB IDs - these may need adjustment for different boards
                const filters = [
                    // Arduino Uno
                    { vendorId: 0x2341, productId: 0x0043 },
                    { vendorId: 0x2341, productId: 0x0001 },
                    { vendorId: 0x2A03, productId: 0x0043 },
                    // Arduino Mega
                    { vendorId: 0x2341, productId: 0x0010 },
                    { vendorId: 0x2341, productId: 0x0042 },
                    { vendorId: 0x2A03, productId: 0x0010 },
                    { vendorId: 0x2A03, productId: 0x0042 },
                    // Arduino Leonardo
                    { vendorId: 0x2341, productId: 0x0036 },
                    { vendorId: 0x2341, productId: 0x8036 },
                    { vendorId: 0x2A03, productId: 0x0036 },
                    // Arduino Micro
                    { vendorId: 0x2341, productId: 0x0037 },
                    { vendorId: 0x2341, productId: 0x8037 },
                    { vendorId: 0x2A03, productId: 0x0037 },
                    // Arduino Nano
                    { vendorId: 0x2341, productId: 0x0043 },
                    { vendorId: 0x1A86, productId: 0x7523 }, // CH340 driver
                    // Add more as needed
                ];
                
                // Request device
                port = await navigator.usb.requestDevice({ filters });
                
                log(`Device selected: ${port.productName}`);
                await port.open();
                
                // Update UI
                connectButton.disabled = true;
                disconnectButton.disabled = false;
                flashSection.classList.remove('hidden');
                statusText.textContent = `Connected to ${port.productName}`;
                connectionStatus.classList.remove('status-disconnected');
                connectionStatus.classList.add('status-connected');
                
                // Update steps
                step1.classList.remove('active');
                step1.classList.add('completed');
                step2.classList.add('active');
                
                log('Connection established successfully', 'success');
                log('Ready to flash firmware', 'info');
            } catch (error) {
                if (error.name === 'NotFoundError') {
                    log('No compatible device selected.', 'error');
                } else {
                    log(`Connection error: ${error.message}`, 'error');
                    console.error(error);
                }
            }
        });
        
        // Disconnect from Arduino
        disconnectButton.addEventListener('click', async () => {
            if (port) {
                try {
                    await port.close();
                    port = null;
                    
                    // Update UI
                    connectButton.disabled = false;
                    disconnectButton.disabled = true;
                    flashSection.classList.add('hidden');
                    statusText.textContent = 'No device connected';
                    connectionStatus.classList.remove('status-connected');
                    connectionStatus.classList.add('status-disconnected');
                    
                    // Update steps
                    step1.classList.add('active');
                    step1.classList.remove('completed');
                    step2.classList.remove('active');
                    step2.classList.remove('completed');
                    
                    log('Device disconnected', 'info');
                } catch (error) {
                    log(`Disconnect error: ${error.message}`, 'error');
                    console.error(error);
                }
            }
        });
        
        // Flash firmware
        flashButton.addEventListener('click', async () => {
            if (!port) {
                return;
            }
            
            const boardType = boardTypeSelect.value;
            
            try {
                log(`Starting upload to ${boardType}...`);
                flashButton.disabled = true;
                
                try {
                    // Create avrgirl instance
                    const avrgirlOptions = {
                        board: boardType,
                        debug: true
                    };
                    
                    // When using WebUSB
                    if (window.AvrgirlArduino && window.AvrgirlArduino.use) {
                        window.AvrgirlArduino.use.usb(port);
                        avrgirlOptions.port = port;
                    }
                    
                    const avrgirl = new window.AvrgirlArduino(avrgirlOptions);
                    
                    // Set up logging
                    avrgirl.on('log', message => {
                        log(message);
                    });
                    
                    avrgirl.on('error', error => {
                        log(`Upload error: ${error.message}`, 'error');
                        flashButton.disabled = false;
                    });
                    
                    // Upload the hex file
                    avrgirl.flash(preloadedHex, (error) => {
                        if (error) {
                            log(`Upload failed: ${error.message}`, 'error');
                            flashButton.disabled = false;
                        } else {
                            log('Upload completed successfully!', 'success');
                            flashButton.disabled = false;
                            
                            // Update steps
                            step2.classList.remove('active');
                            step2.classList.add('completed');
                        }
                    });
                } catch (err) {
                    log(`Upload preparation failed: ${err.message}`, 'error');
                    flashButton.disabled = false;
                }
            } catch (error) {
                log(`Upload preparation error: ${error.message}`, 'error');
                flashButton.disabled = false;
            }
        });
        
        // Initialize
        log('Arduino Web Flasher initialized');
        log('Please connect your Arduino board by clicking the Connect button.');
        
        // Check for permission to use USB
        navigator.usb.getDevices()
            .then(devices => {
                if (devices.length > 0) {
                    log(`Found ${devices.length} previously connected devices`);
                }
            })
            .catch(error => {
                log(`USB enumeration error: ${error.message}`, 'error');
            });
    </script>
</body>
</html>
