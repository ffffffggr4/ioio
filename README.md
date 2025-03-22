<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Encrypted Chat</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; }
        #chat { height: 300px; overflow-y: scroll; border: 1px solid #ccc; padding: 10px; }
        .message { margin-bottom: 10px; }
    </style>
</head>
<body>
    <h1>Encrypted Chat</h1>

    <div id="chat"></div>

    <form id="message-form">
        <input type="text" id="message" placeholder="Type your message" required>
        <input type="text" id="encryption-key" placeholder="Enter encryption key" required>
        <button type="button" id="generate-key">Generate Key</button>
        <button type="submit">Send</button>
    </form>

    <br>
    <label for="decryption-key">Decryption Key:</label>
    <input type="text" id="decryption-key" placeholder="Enter decryption key">
    <button id="decrypt-button">Decrypt Messages</button>

    <script>
        $('#generate-key').on('click', async function() {
            try {
                const key = await generateEncryptionKey();
                $('#encryption-key').val(key);
            } catch (error) {
                alert('Failed to generate encryption key: ' + error.message);
            }
        });

        async function generateEncryptionKey() {
            const text = prompt("Enter a word or phrase to use as a key:");
            if (!text) {
                throw new Error("Text for key generation is required");
            }

            const response = await fetch(`/generate_key?text=${encodeURIComponent(text)}`);
            if (!response.ok) {
                throw new Error('Failed to generate key. Server returned ' + response.status);
            }
            const data = await response.json();
            return data.key;
        }

        function fetchMessages() {
            const decryptionKey = $('#decryption-key').val();
            $.get('/get_messages', { decryption_key: decryptionKey }, function(data) {
                const chat = $('#chat');
                chat.empty();
                data.messages.forEach(msg => {
                    chat.append('<div class="message">' + msg + '</div>');
                });
            });
        }

        $(document).ready(function() {
            $('#message-form').on('submit', function(e) {
                e.preventDefault();
                const message = $('#message').val();
                const encryptionKey = $('#encryption-key').val();

                $.post('/send_message', { message: message, encryption_key: encryptionKey }, function(data) {
                    if (data.status) {
                        $('#message').val('');
                        fetchMessages();
                    } else {
                        alert(data.error);
                    }
                });
            });

            $('#decrypt-button').on('click', function() {
                fetchMessages();
            });

            fetchMessages();
        });
    </script>
</body>
</html>
