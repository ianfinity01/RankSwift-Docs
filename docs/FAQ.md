## Does the server use any form of encryption for requests?
**No.** Encryption would require the user to set up SSL certificates for HTTPS, which is rather difficult for the average user and greately complicates things. In the future, we may allow you to add an SSL certificate to your server for an encrypted connection, however, encryption **should not be an issue**. A man in the middle attack is highly unlikely considering communication is only happening from a roblox server to your hosting service, not from client's computers to the hosting service.

## Generally, what can the server handle?
From some preliminary testing, it seems that it should be able to easily handle **25 million entries** and **> 1000 requests per second**, however the latter varies greately depending on your network bandwidth and your processing power. The board size can also be extended based on your situation, but it can be a bit difficult. It is advised you run some [stress testing](./Server/stress-testing.md) with the system on your hosting service.

## Don't see your question here?
Email me at [ianfinity1@gmail.com](mailto:ianfinity1@gmail.com)! I'll answer your question and maybe even put it on here.