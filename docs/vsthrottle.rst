===============
vmod_vsthrottle
===============

-------------------------
Varnish Throttling Module
-------------------------

SYNOPSIS
========

import vsthrottle;

DESCRIPTION
===========

A Varnish vmod for rate-limiting traffic on a single Varnish
server. Offers a simple interface for throttling traffic on a per-key
basis to a specific request rate.

Keys can be specified from any VCL string, e.g. based on client.ip, a
specific cookie value, an API token, etc.

The request rate is specified as the number of requests permitted over
a period. To keep things simple, this is passed as two separate
parameters, 'limit' and 'period'.

This VMOD implements a `token-bucket algorithm`_. State associated
with the token bucket for each key is stored in-memory using BSD's
red-black tree implementation.

Memory usage is around 100 bytes per key tracked.

.. _token-bucket algorithm: http://en.wikipedia.org/wiki/Token_bucket


FUNCTIONS
=========

is_denied
---------

Prototype
        ::

                is_denied(STRING key, INT limit, DURATION period)
Arguments
	key: A unique identifier to define what is being throttled - more examples below
	
	limit: How many requests in the specified period
	
	period: The time period
	
Return value
	BOOL
Description
	Can be used to rate limit the traffic for a specific key to a
	maximum of 'limit' requests per 'period' time. A token bucket
	is uniquely identified by the triplet of its key, limit and
	period, so using the same key multiple places with different
	rules will create multiple token buckets.

Example
        ::

		sub vcl_recv {
			if (vsthrottle.is_denied(client.identity, 15, 10s)) {
				# Client has exceeded 15 reqs per 10s
				return (synth(429, "Too Many Requests"));
			}

			# ...
		}


USAGE
=====

In your VCL you can now use this vmod along the following lines::
        
        import vsthrottle;
        
        sub vcl_recv {
        	if (vsthrottle.is_denied(client.identity, 15, 10s)) {
        		# Client has exceeded 15 reqs per 10s
        		return (synth(429, "Too Many Requests"));
        	}
        }
