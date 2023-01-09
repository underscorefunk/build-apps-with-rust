# Storing data on the client

Web applications have the ability to save data to/in the browser. 

## Available tools

- [Web storage: Session Storage](/client/store_data/web_storage.md)
- [Web storage: Local Storage](/client/store_data/web_storage.md)
- [Cookies](/client/store_data/cookies.md)

## Example usage

- Web Storage
	- Non-sensitive information
	- Application settings 
	- Application state changes for offline usage
- Cookies
	- Session or user information that may change the server's respose

## Comparison

|  | Session Storage | Local Storage | Cookies |
|-|------------------|---------------|---------|
| Deleted when browser data is cleared | ✅ | ✅ | ✅ |
| Can be modified outside of your application | ✅ | ✅ | ✅ |
| Deleted when browser is closed | ✅ | | |
| Sent with every web request | | | ✅ |

## Caveats

**Persistance:** We can not always guarantee that data stored on the client (in the browser) will persist. Users are in control of clearing browser caches and data stores.

**Security:** Local storage and cookie data can be easily read by anyone using a web brower's development tools. There are no restrictions preventing third part scripts from accessing local storage or cookies as well. 

**Guarantees:** The lack of persistance and security means that we should not assume integrity of data stored in the client. 

## Additional resources

- [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) 
- [Cookies Api](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/cookies)