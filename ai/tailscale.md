# 📘 Tailscale Network

## 🔹 Step 1 : Installation on Clients

1. **Go to https://tailscale.com/ and sign in with Microsoft, Google or other**

2. **Download the Tailscale client on one of the client machines (laptop, smartphone)**

3. **After installation, sign in with the same account**

Repeat the operation for each client machine to connect

## 🔹 Step 2 : Installation on the Server

1. **Open an SSH session to the server**

2. **Start installation with the official script**

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

3. **Start the network**

```bash
sudo tailscale up
```

4. **Open the displayed link with a web browser, and sign in with the same account as in step 1**

5. **Enable HTTPS exposure on the Tailscale administration console**

Go to the [**DNS** tab](https://login.tailscale.com/admin/dns), in the **HTTPS Certificates** section, click on **Enable https...**

6. **Expose Open WebUI over HTTPS**

```bash
sudo tailscale serve --bg localhost:3000
```

7. **Verify functionality from one of the client machines**

Retrieve the Linux server name from the administration console, and type its name in the address bar of a browser from one of the client machines. For example `https://aiserver.tail123456789.ts.net/`


## 🔹 Step 3 : Strengthen Security

This operation aims to allow only the necessary traffic within the Tailscale network.

1. **Create a tag named `server`**

Go to the [**Access control** then **Tags** tab](https://login.tailscale.com/admin/acls/visual/tags). Click on **Create tag**, and enter `server` and validate.

2. **Assign the `server` tag to the Linux server**

Go to the [**Machines** tab](https://login.tailscale.com/admin/machines), at the level of the Linux server, click on **...** then **Edit ACL tags...**. Enter `server` and validate.

3. **Modify default rules to allow only traffic to the server**

Return to the [**Access control** tab](https://login.tailscale.com/admin/acls/visual/general-access-rules). On the only rule, click on **...** then **Edit**.

Remove the value in **destination**, and enter `tag:server` instead.

Remove the value in **Port and protocol**, and enter `tcp:443`, then `tcp:22` then `tcp:11434` instead.

The expected result is :
```json
// Allow access only to servers (ssh, https and ollama)
{
	"src": ["*"],
	"dst": ["tag:server"],
	"ip":  ["tcp:443", "tcp:11434", "tcp:22"]
}
```

Save with **Save grant**
