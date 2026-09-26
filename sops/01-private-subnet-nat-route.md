## SOP: Diagnose and Fix Private Subnet Internet Access via Route Table and NAT Gateway

### Objective

This SOP explains how to inspect a private subnet’s route table, confirm the NAT gateway is healthy, add the missing default route to the NAT gateway, and verify end-to-end internet connectivity from the subnet. Follow these steps to restore outbound internet access for private subnet workloads.

---

### Key Steps

**1. Retrieve the Private Subnet ID** [0:18](https://loom.com/share/150b4ee6c4e34066b282a47dbc943518?t=18)

<img width="1280" height="720" alt="Private Subnet SOP 1" src="https://github.com/user-attachments/assets/ed35eabe-8416-46de-8869-d6338c89149c" />


- Open the AWS CLI terminal.
- Run the AWS `describe` command to retrieve the subnet ID for the private subnet.
- Paste in the API server IP address when prompted.
- Confirm the command returns the subnet ID you will use in the next step.

---

**2. Inspect the Route Table for the Subnet** [1:46](https://loom.com/share/150b4ee6c4e34066b282a47dbc943518?t=106)

<img width="1280" height="720" alt="Private Subnet SOP 2" src="https://github.com/user-attachments/assets/d5e2226f-d84e-421b-aa49-b47aea22044c" />


- Use the subnet ID from the previous step in a new AWS `describe` command.
- Include the full subnet identifier exactly as returned, including the `subnet-` prefix.
- Review the route table output.
- If the output is paginated, continue scrolling/down until the full route table details are visible.
- Check the route table ID, VPC ID, associations, and route state.

---

**3. Determine Whether a Default Internet Route Exists** [2:33](https://loom.com/share/150b4ee6c4e34066b282a47dbc943518?t=153)

<img width="1280" height="720" alt="Private Subnet SOP 3" src="https://github.com/user-attachments/assets/a1b3895a-fb7d-400c-b0af-621cdbc7b566" />


- Review the route table entries for a default route (`0.0.0.0/0`).
- Confirm whether traffic destined for the public internet is routed to a NAT gateway.
- If the route table only contains routes for the VPC/private network, outbound internet access is not configured.
- Note that private subnets do not have direct internet access by default.

---

**4. Verify the NAT Gateway Is Healthy** [3:42](https://loom.com/share/150b4ee6c4e34066b282a47dbc943518?t=222)

<img width="1280" height="720" alt="Private Subnet SOP 4" src="https://github.com/user-attachments/assets/627ffdda-664c-4bde-bfb3-383bd0e5ca3c" />


- Run an AWS `describe` command for the NAT gateway.
- Confirm the NAT gateway status is available/healthy.
- Verify the status shows success and that the gateway is ready to accept traffic.
- If the NAT gateway is not healthy, resolve that issue before changing the route table.

---

**5. Create the Missing Route to the NAT Gateway** [5:00](https://loom.com/share/150b4ee6c4e34066b282a47dbc943518?t=300)

<img width="1280" height="720" alt="Private Subnet SOP 5" src="https://github.com/user-attachments/assets/3c36a92f-2147-4c37-bb33-ee5748a8c73f" />


- Return to the AWS CLI terminal.
- Gather the NAT gateway ID.
- Gather the route table ID for the private subnet.
- Create a new route in the route table with: 
  - Destination: `0.0.0.0/0`
  - Target: the NAT gateway ID
- Confirm the command returns success (for example, `true`).

---

**6. Confirm the Route Was Added Correctly** [6:44](https://loom.com/share/150b4ee6c4e34066b282a47dbc943518?t=404)

<img width="1280" height="720" alt="Private Subnet SOP 6" src="https://github.com/user-attachments/assets/0f1bd0f9-d064-40ce-8ff2-f5a2a4c5a29b" />

- Review the route table output again.
- Verify the new route exists and points all public traffic to the NAT gateway.
- Confirm the destination is `0.0.0.0/0`.
- Confirm the NAT gateway ID is listed as the target.
- Ensure the subnet now has a path for outbound internet traffic.

---

**7. Validate the Fix End to End** [7:32](https://loom.com/share/150b4ee6c4e34066b282a47dbc943518?t=452)


<img width="1280" height="720" alt="Private Subnet SOP 7" src="https://github.com/user-attachments/assets/2ed603ce-6f7a-4807-a7c6-1bbd8965a941" />


- Run the validation script (`validate.sh`) to check the environment.
- Perform an actual network request using `curl` to test the full connection path.
- Confirm the TCP/TLS handshake succeeds.
- Verify the host resolves and the request reaches the public endpoint successfully.
- Treat a successful request as confirmation that the private subnet internet access issue is resolved.

---

### Cautionary Notes

- Do not create the route until you have confirmed the NAT gateway is healthy and available.
- Make sure you copy the full subnet ID and route table ID exactly as returned, including prefixes such as `subnet-`.
- If the CLI output is truncated or paginated, scroll through the full output before making changes.
- A private subnet will not have direct internet access unless a default route to a NAT gateway is explicitly configured.
- Validate with an actual network request, not just by checking configuration, to confirm end-to-end connectivity.

---

### Tips for Efficiency

- Keep the subnet ID, route table ID, and NAT gateway ID in a scratch pad before making changes.
- Reuse the same AWS CLI session to avoid re-authentication or context switching.
- Check the route table before and after the change so you can quickly confirm what was missing.
- Use `curl` or a similar request as the final verification step to catch DNS, routing, and TLS issues in one test.
- If multiple private subnets use the same route table, one route update may restore connectivity for all associated subnets.

---

### Link to Loom

<https://loom.com/share/150b4ee6c4e34066b282a47dbc943518>
