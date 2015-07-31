# GSS-APIv3

GSS-APIv3, made easier and extended to TLS and such (all currently vapourware)

The GSS-APIv2u1 is a bit of a mess.  It's time to cleanup.

Why GSS at all?  Well, with an opaque-ish approach to naming it has a
great deal of promise as a simpler API for TLS, not just Kerberos.

This README is barebones.  It assumes the reader has some familarity
with the GSS-API.  Note the lack of links for the time being.

## Is This Worthwhile?

Maybe not.  Consider SSPI: it covers GSS/Kerberos, NTLMSSP, SASL, and
TLS (SChannel).  Clearly that's quite useful.  But if it's so useful,
maybe we should build a portable SSPI and to hell with GSS-API (though
we might initially backend a portable SSPI with portable GSS
implementations).

I tend to think that a clean API would be much better in terms of a)
getting adoption, and b) getting people to use TLS and Kerberos
correctly.  (b) is a big motivator.

## GSS-APIv2u1 C-Bindings Pain Points

 - Incomplete ABI for C.
 - DLL Hell in the presence of multiple implementations.
 - A great number of functions, many with a large number of arguments.
   Yes, common applications use a very small number of these functions,
   so some of this is a documentation problem, but nonetheless we can
   improve things.
 - gss_init_sec_context() and gss_accept_sec_context() are horrible.
 - gss_display_status() is horrible.
 - ...

## Design Principles and Ideas for the New C-Bindings

Zeroth, the new API should be possible to backend with the old one, so
that we can get a new implementation up and running quickly.  This is
important as we can't build Rome in one day.  This has to be a shim over
GSS-APIv2u1 just as it is possible (and has been done) to build a
GSS-API shim over SSPI.

First, by and large the treatment of NAME, CREDENTIAL HANDLE, and
SECURITY CONTEXT as opaque objects in the abstract API and C and Java
bindings has been a win.  This won't go away.

The use of OIDs has been a usability disaster, even when there are
symbols that resolve to the OIDs one needs, there are still output OIDs
and OID SETs to deal with.  OIDs must go.  Instead we must use URNs,
though still with symbols available for all the identifiers a developer
is likely to need.

To the extent that the GSS-APIv2u1 is easy to use, the new version
should not be harder.  The new version should be easier to use across
the board.

Second, the gss_init/accept_sec_context() mess should be replaced with a
trivial function that creates a shell/empty security context, and
accessors for all the arguments of those functions except the
input/output tokens (which should be used with a "stepper" function).

This is a principle that should apply in general: simple functions, with
few arguments, though perhaps many accessor functions that can be easily
segregated as far as documentation goes.

Third, to deal with the DLL Hell issues, the API should involve
structures with function pointer tables so as to make it easy to ensure
that a single backend is used with a related set of GSS objects, even
when passing them across library boundaries.  The trick here is
extensibility -- C is not C++, after all, but I believe it can be done,
though it will require a common registry (in the IANA registry sense,
though not necessarily IANA).

Remember, passing and returning structures by value is an option, and
not a bad one!

Fourth, memory management has to be as automatic as possible.  For
example, there should be no need to release a name object that is
obtained from a credential handle or a security context.

And different types must be used for byte buffers allocated by the API
versus ones that are input to the API by the application: so that the
application does not accidentally use a GSS release function to release
buffers allocated by the application.

## Design Points

TBD.

 - There will be a single entry point into the API that returns an
   object that is akin to a krb5_context in the krb5 C APIs.  This will
   replace the awful `OM_uint32 *minor_status` argument, and won't be
   necessary in functions where other inputs (e.g., a credential handle)
   can supply it.

