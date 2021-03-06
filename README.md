clipcaptcha
===========

A Tool for Impersonating CAPTCHA Providers

**Author**: Gursev Singh Kalra (gursev.kalra@foundstone.com - McAfee, Foundstone)

**clipcaptcha** is an extensible and signature based CAPTCHA Provider impersonation tool based off Moxie Marlinspike's sslstrip codebase. For additional information please visit: http://www.foundstone.com and http://blog.opensecurityresearch.com 

It requires Python 2.5 or newer, along with the 'twisted' python module.

**Running clipcaptcha**:
	
	Run 'python clipcaptcha.py -h' for help.

	The four steps to getting this working on Linux are:

	1. Enable forwarding mode on your machine 
		echo "1" > /proc/sys/net/ipv4/ip_forward

	2. Setup iptables to redirect HTTP traffic to clipcaptcha. 
		iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port <listeningPort>

	3. Run arpspoof to redirect the traffic to your machine. 
		arpspoof -i <interface> -t <targetIP> <gatewayIP>

	4. Run clipcaptcha in one of its mode of operation. 
		clipcaptcha.py <mode> -p <listeningPort>

		clipcaptcha can be run in one of the following five operational modes and selecting
		more than one operation mode is an error:
		a. Monitor Mode: Signature based CAPTCHA provider detection is performed 
		   and all CAPTCHA validation requests are logged to a local file. The CAPTCHA 
		   validation requests and corresponding responses are allowed to complete without 
		   any modifications.
		b. Avalanche Mode: Success response is returned on the matching CAPTCHA provider 
		   for all validation requests. It is recommended to not run clipcaptcha in this 
		   mode as a surge in successful account creation or registrations may be detected.
		c. Stealth Mode: Stealth is the recommended mode for running clipcaptcha. This 
		   mode relies on the fact that all CAPTCHA validation API.s need to send user 
		   supplied CAPTCHA solution to the CAPTCHA providers for validation. 
		   clipcaptcha banks on this behavior to operate stealthily and return Success 
		   status only for the requests that contain a secret string. In its current 
		   implementation, clipcaptcha parses the entire CAPTCHA validation request 
		   (initial line, headers and body) and returns success if the secret string is 
		   found or allows the request to complete without any modifications.
		d. DoS Mode: Failure response is returned for all CAPTCHA validation requests. 
		   This leads to a Denial of Service condition on the target web application 
		   for all forms that require CAPTCHA validation.
		e. Random Mode: Random Success and Failure responses are returned as per the 
		   matching CAPTCHA provider for all validation requests and exits only 
		   as a teaser mode.

	Once clipcaptcha instance starts running, all CAPTCHA validation requests will be 
	administered by clipcaptcha.

**Signature based CAPTCHA provider detection**:

All CAPTCHA providers are basically HTTP based custom web services. These services accept CAPTCHA validation requests in a particular format and respond with finite set of responses that allow the clients to make Boolean choices to allow or disallow the request. clipcaptcha takes advantage of this finite and predictable request and response data set to implement signature based request detection and response system. For additional details please refer clipcaptcha whitepaper. The default configuration contains signatures for reCAPTCHA, opencaptcha and captchator.


	Example runs:
	clipcaptcha.py -m -p 7777 	# Run clipcaptcha in monitor mode listening on port 7777.

	clipcaptcha.py -a -p 8888 	# Run clipcaptcha in avalanche mode and approve all CAPTCHAs 
					# for which signatures are available) 

	clipcaptcha.py -s clipcaptcha 	# Run clipcaptcha in stealth mode and approve all CAPTCHA 
					# requests which contain the secret string "clipcaptcha"
