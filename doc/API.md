# PageKite API reference manual

   * Initialization
      * [`pagekite_init                               `](#pgktnt)
      * [`pagekite_init_pagekitenet                   `](#pgktntpgktnt)
      * [`pagekite_init_whitelabel                    `](#pgktntwhtlbl)
      * [`pagekite_add_kite                           `](#pgktddkt)
      * [`pagekite_add_service_frontends              `](#pgktddsrvcfrntnds)
      * [`pagekite_add_whitelabel_frontends           `](#pgktddwhtlblfrntnds)
      * [`pagekite_lookup_and_add_frontend            `](#pgktlkpndddfrntnd)
      * [`pagekite_add_frontend                       `](#pgktddfrntnd)
      * [`pagekite_set_log_mask                       `](#pgktstlgmsk)
      * [`pagekite_set_housekeeping_min_interval      `](#pgktsthskpngmnntrvl)
      * [`pagekite_set_housekeeping_max_interval      `](#pgktsthskpngmxntrvl)
      * [`pagekite_enable_http_forwarding_headers     `](#pgktnblhttpfrwrdnghdrs)
      * [`pagekite_enable_fake_ping                   `](#pgktnblfkpng)
      * [`pagekite_enable_watchdog                    `](#pgktnblwtchdg)
      * [`pagekite_enable_tick_timer                  `](#pgktnbltcktmr)
      * [`pagekite_set_conn_eviction_idle_s           `](#pgktstcnnvctndls)
      * [`pagekite_set_openssl_ciphers                `](#pgktstpnsslcphrs)
      * [`pagekite_want_spare_frontends               `](#pgktwntsprfrntnds)
   * Lifecycle
      * [`pagekite_thread_start                       `](#pgktthrdstrt)
      * [`pagekite_thread_wait                        `](#pgktthrdwt)
      * [`pagekite_thread_stop                        `](#pgktthrdstp)
      * [`pagekite_free                               `](#pgktfr)
      * [`pagekite_get_status                         `](#pgktgtstts)
      * [`pagekite_get_log                            `](#pgktgtlg)
      * [`pagekite_poll                               `](#pgktpll)
      * [`pagekite_tick                               `](#pgkttck)
      * [`pagekite_set_bail_on_errors                 `](#pgktstblnrrrs)
      * [`pagekite_perror                             `](#pgktprrr)
   * Experimental
      * [`pagekite_add_listener                       `](#pgktddlstnr)
      * [`pagekite_enable_lua_plugins                 `](#pgktnbllplgns)
   * [Constants](#constants)

## Functions

### Initialization

<a                                                       name="pgktnt"><hr></a>

#### `pagekite_mgr pagekite_init(...)`

Initialize the PageKite manager.

This allocates a static amount of RAM for PageKite (not counting
buffers managed by OpenSSL). Since libpagekite's resource usage
is allocated up-front, you need to specify maximum numbers of
kites, front-end relays and in-flight connections you want to
keep track of at any one time.

The `flags` variable should be used with the constants `PK_WITH_*`
and `PK_AS_*`, bitwise OR'ed together to tune the behaviour of
libpagekite. To use the recommended defaults, simply specify `PK_WITH_DEFAULTS`.
Requesting defaults is a forward compatible choice and will continue
to use recommended settings even as new features are added to
the library.

The `verbosity` argument controls the internal logging. A small
integer (-1, 0, 1, 2) can be used to choose from a predefined
level, for more fine-grained control use the `PK_LOG_` constants
bitwise OR'ed together to enable logging of individual subsystems.

This method can only be called before starting the master thread.

**Arguments**:

   * `const char* app_id`: Short ID, reported in instrumentation
   * `int max_kites`: Max number of kite names allowed
   * `int max_frontends`: Max number of front-end relays recognized
   * `int max_conns`: Max number of in-flight connections
   * `const char* dyndns_url`: Dynamic DNS update URL (format string)
   * `int flags`: Flags, which features to enable
   * `int verbosity`: Verbosity or log mask

**Returns**: A reference to the PageKite manager object.


<a                                                 name="pgktntpgktnt"><hr></a>

#### `pagekite_mgr pagekite_init_pagekitenet(...)`

Initialize the PageKite manager, configured for use with the pagekite.net
public service.

See the basic init docs for further details.

If `flags` do not include `PK_WITHOUT_SERVICE_FRONTENDS`, service
frontends will be chosen automatically using the given domain
name.

This method can only be called before starting the master thread.

**Arguments**:

   * `const char* app_id`: Short ID, reported in instrumentation
   * `int max_kites`: Max number of kite names allowed
   * `int max_conns`: Max number of in-flight connections
   * `int flags`: Flags, which features to enable
   * `int verbosity`: Verbosity or log mask

**Returns**: A reference to the PageKite manager object.


<a                                                 name="pgktntwhtlbl"><hr></a>

#### `pagekite_mgr pagekite_init_whitelabel(...)`

Initialize the PageKite manager, configured for use with a pagekite.net
white-label domain.

See the basic init docs for further details.

If `flags` do not include `PK_WITHOUT_SERVICE_FRONTENDS`, white-label
frontends will be chosen automatically using the given domain
name.

This method can only be called before starting the master thread.

**Arguments**:

   * `const char* app_id`: Short ID, reported in instrumentation
   * `int max_kites`: Max number of kite names allowed
   * `int max_conns`: Max number of in-flight connections
   * `int flags`: Flags, which features to enable
   * `int verbosity`: Verbosity or log mask
   * `const char* whitelabel_tld`: Top level domain of white-label kites

**Returns**: A reference to the PageKite manager object.


<a                                                     name="pgktddkt"><hr></a>

#### `int pagekite_add_kite(...)`

Configure a "kite", mapping a public domain to a local service.

Multiple kites can be configured for the same domain name, by
calling this function multiple times, as long as the public port
or protocol differ.

This method can only be called before starting the master thread.

Note: When used with the "raw" protocol, the public port cannot
be 0.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `const char* proto`: Protocol
   * `const char* kitename`: Kite DNS name
   * `int pport`: Public port, 0 for default/any
   * `const char* secret`: Kite secret for authentication
   * `const char* backend`: Hostname of the origin server
   * `int lport`: Port of the origin server

**Returns**: 0 on success, -1 on failure.


<a                                            name="pgktddsrvcfrntnds"><hr></a>

#### `int pagekite_add_service_frontends(...)`

Configure libpagekite to use the Pagekite.net pool of public front-end
relay servers.

This method can only be called before starting the master thread.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int flags`: Flags to enable IPv4, IPv6 and/or dynamic frontends

**Returns**: The number of relay IPs configured, or -1 on failure.


<a                                          name="pgktddwhtlblfrntnds"><hr></a>

#### `int pagekite_add_whitelabel_frontends(...)`

Configure libpagekite to use the relays associated with a Pagekite.net
white-label domain.

This method can only be called before starting the master thread.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int flags`: Flags to enable IPv4, IPv6 and/or dynamic frontends
   * `const char* whitelabel_tld`: Top level domain of white-label kites

**Returns**: The number of relay IPs configured, or -1 on failure.


<a                                            name="pgktlkpndddfrntnd"><hr></a>

#### `int pagekite_lookup_and_add_frontend(...)`

Configure libpagekite front-end relays.

All available IP addresses referenced by the specified DNS name
will be configured as potential relays, and the app will choose
between them based on performance metrics.

If `update_from_dns` is nonzero, DNS will be rechecked periodically
for new relay IPs. Enabling such updates is generally preferred
over a completely static configuration, so long-running instances
can keep up with changes to the relay infrastructure.

This method can only be called before starting the master thread.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `const char* domain`: DNS name of the frontend (or pool of frontends)
   * `int port`: Port to connect to
   * `int update_from_dns`: Set nonzero to re-lookup periodically from DNS

**Returns**: The number of relay IPs configured, or -1 on failure.


<a                                                 name="pgktddfrntnd"><hr></a>

#### `int pagekite_add_frontend(...)`

Configure libpagekite front-end relays.

All available IP addresses referenced by the specified DNS name
will be configured as potential relays, and the app will choose
between them based on performance metrics.

This method is static - DNS is only checked on startup. This is
not optimal and this method is only provided for backwards compatibility.
New apps should enable periodic DNS updates.

This method can only be called before starting the master thread.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `const char* domain`: DNS name of the frontend (or pool of frontends)
   * `int port`: Port to connect to

**Returns**: The number of relay IPs configured, or -1 on failure.


<a                                                  name="pgktstlgmsk"><hr></a>

#### `int pagekite_set_log_mask(...)`

Configure the log verbosity using a bitmask.

See the `PK_LOG_*` constants for options.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int mask`: A bitmask

**Returns**: Always returns 0.


<a                                          name="pgktsthskpngmnntrvl"><hr></a>

#### `int pagekite_set_housekeeping_min_interval(...)`

Configure the minimum interval for internal housekeeping.

Internal housekeeping includes attempting to re-establish a connection
to the required relays, occasionally refreshing information from
DNS and pinging tunnels to detect whether they have gone silently
dead (due to NAT timeouts, for example).

Setting this interval too low may reduce battery life or increase
load on shared infrastructure, as a result there is a hard-coded
minimum value which this function cannot override. Setting this
interval too high may prevent libpagekite from detecting network
outages and recovering in a timely fashion.

The app will by default choose an interval between the minimum
and maximum as appropriate. Most apps will not need to change
this setting.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int interval`: Interval, in seconds

**Returns**: The new minimum housekeeping interval.


<a                                          name="pgktsthskpngmxntrvl"><hr></a>

#### `int pagekite_set_housekeeping_max_interval(...)`

Configure the maximum interval for internal housekeeping.

See the documentation for the minimum housekeeping interval for
details and performance concerns.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int interval`: Interval, in seconds

**Returns**: The new maximum housekeeping interval.


<a                                       name="pgktnblhttpfrwrdnghdrs"><hr></a>

#### `int pagekite_enable_http_forwarding_headers(...)`

Enable or disable HTTP forwarding headers.

When enabled, libpagekite will rewrite incoming HTTP headers to
add information about the remote IP address and remote protocol.

The `X-Forwarded-For` header will contain the IP address of the
remote HTTP client, as reported by the front-end relay (probably
in IPv6 notation).

The `X-Forwarded-Proto` header will report whether the connection
to the relay was HTTP or HTTPS. This can be detected and used
to redirect or reject plain-text connections.

**Limitations**: Headers are only added to the first request of
a persistent HTTP/1.1 session. The data reported comes from the
front-end relay and a malicious relay could provide false data.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int enable`: 0 disables, any other value enables

**Returns**: Always returns 0.


<a                                                 name="pgktnblfkpng"><hr></a>

#### `int pagekite_enable_fake_ping(...)`

Enable or disable fake pings.

This is a debugging/testing option, which effectively randomizes
which front-end relay is used and increases the frequency of migrations.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int enable`: 0 disables, any other value enables

**Returns**: Always returns 0.


<a                                                name="pgktnblwtchdg"><hr></a>

#### `int pagekite_enable_watchdog(...)`

Enable or disable watchdog.

The watchdog is a thread which periodically checks if the main
pagekite thread has locked up. If it thinks the app has locked
up, it will cause a segmentation fault, which in turn will create
a core dump for debugging (assuming ulimits allow).

This method can only be called before starting the master thread.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int enable`: 0 disables, any other value enables

**Returns**: Always returns 0.


<a                                                name="pgktnbltcktmr"><hr></a>

#### `int pagekite_enable_tick_timer(...)`

Enable or disable tick event timer.

This method can be used to toggle the tick event timer on or off
(it is on by default). Apps may want to disable the timer for
power saving reasons.

If the timer is disabled, the tick method should be called instead
when possible, to ensure that housekeeping takes place.

This method may be called at any time, but may hang as it waits
for the main event-loop lock.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int enable`: 0 disables, any other value enables

**Returns**: Always returns 0.


<a                                             name="pgktstcnnvctndls"><hr></a>

#### `int pagekite_set_conn_eviction_idle_s(...)`

Configure eviction of idle connections.

As libpagekite works with a fixed pool of RAM, it may be unable
to allocate buffers for new incoming connections. When this happens,
the oldest connection which has been idle for more than `seconds`
seconds may be evicted.

Set `seconds = 0` to disable eviction and instead reject incoming
connections when overloaded. Eviction is disabled by default.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int seconds`: Minimum idle time to qualify for eviction

**Returns**: Always returns 0.


<a                                             name="pgktstpnsslcphrs"><hr></a>

#### `int pagekite_set_openssl_ciphers(...)`

Choose which ciphers to use in TLS

See the SSL_set_cipher_list(3) and ciphers(1) man pages for details.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `const char* ciphers`: The string passed to OpenSSL

**Returns**: Always returns 0.


<a                                            name="pgktwntsprfrntnds"><hr></a>

#### `int pagekite_want_spare_frontends(...)`

Connect to multiple front-end relays.

If non-zero, this setting will configure how many spare relays
to connect to at any given time. This may increase availability
or performance in some special cases, but increases the load on
shared relay infrastructure and should be avoided if possible.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int spares`: Number of spare front-ends to connect to

**Returns**: Always returns 0.

### Lifecycle

<a                                                 name="pgktthrdstrt"><hr></a>

#### `int pagekite_thread_start(...)`

Start the main thread: run pagekite!

This function should only be called once (per session).

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object

**Returns**: The return value of `pthread_create()`


<a                                                   name="pgktthrdwt"><hr></a>

#### `int pagekite_thread_wait(...)`

Wait for the main pagekite thread to finish.

This function should only be called once (per session).

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object

**Returns**: The return value of `pthread_join()`


<a                                                  name="pgktthrdstp"><hr></a>

#### `int pagekite_thread_stop(...)`

Stop the main pagekite thread.

This function should only be called once (per session).

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object

**Returns**: The return value of `pthread_join()`


<a                                                       name="pgktfr"><hr></a>

#### `int pagekite_free(...)`

Free the internal libpagekite buffers.

Call this to free any memory allocated by the init functions.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object

**Returns**: 0 on success, -1 on failure.


<a                                                   name="pgktgtstts"><hr></a>

#### `int pagekite_get_status(...)`

Get the current status of the app.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object

**Returns**: A `PK_STATUS_*` code.


<a                                                     name="pgktgtlg"><hr></a>

#### `char* pagekite_get_log(...)`

Fetch the in-memory log buffer.

Note that the C-API version returns a pointer to a static buffer.
Subsequent calls will overwrite with new data.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object

**Returns**: A snapshot of the current log status.


<a                                                      name="pgktpll"><hr></a>

#### `int pagekite_poll(...)`

Wait for the pagekite event loop to wake up.

This method can be used to pause a thread, waking up again when
libpagekite status changes, network activity takes place or other
events take place which might affect the log or state variable.

This method can be called any time the main thread is running.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int timeout`: Max time to wait, in seconds

**Returns**: 0 on success, -1 if unconfigured.


<a                                                      name="pgkttck"><hr></a>

#### `int pagekite_tick(...)`

Manually trigger the internal tick event.

The internal tick event may trigger housekeeping if necessary.

This method is only needed if the internal periodic timer has
been disabled (e.g. on a mobile app, to conserve battery life).

This method can be called any time the main thread is running.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object

**Returns**: 0 on success, -1 if unconfigured.


<a                                                name="pgktstblnrrrs"><hr></a>

#### `int pagekite_set_bail_on_errors(...)`

Enable or disable bailing out on errors.

If enabled, the app will increase log verbosity and finally call
`exit(100)` after too many errors have occurred. This can help
catch and handle error states, but care should be taken as it
will also bring down the hosting app.

The sensitivity of this function depends on the `errors` variable.
After `errors * 9` problems would have been logged, verbosity
is increased. After `errors * 10`, the app will exit.

This function can be called at any time.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int errors`: An error threshold

**Returns**: Always returns 0.


<a                                                     name="pgktprrr"><hr></a>

#### `void pagekite_perror(...)`

Log an error and reset the internal error state.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `const char*`: Prefix for the logged message

**Returns**: Always returns 0.

### Experimental

<a                                                  name="pgktddlstnr"><hr></a>

#### `int pagekite_add_listener(...)`

Add a listening port to the event loop.

When a connection has been accepted, the requested callback will
be invoked. The callback should take two arguments: the first
an int (the file descriptor of the accepted connection), the second
a `void*` which will contain the `callback_data`.

**Note:** This function is experimental and may change in the
future.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `const char* domain`: Hostname or IP to listen on, e.g. "0.0.0.0"
   * `int port`: Port to listen on
   * `pagekite_callback_t* callback_func`: Callback function
   * `void* callback_data`: Arbitrary data to pass to callback on connect

**Returns**: The number of relay IPs configured, or -1 on failure.


<a                                                name="pgktnbllplgns"><hr></a>

#### `int pagekite_enable_lua_plugins(...)`

Enable and configure Lua plugins.

This will configure and enable Lua plugins, either only those
specified in the `settings` list, or all default plugins if `enable_defaults`
is nonzero.

**Note:** This function is experimental and may change in the
future.

**Arguments**:

   * `pagekite_mgr`: A reference to the PageKite manager object
   * `int enable_defaults`: If non-zero, default plugins will be enabled
   * `char** settings`: NULL-terminated list of settings, e.g. web_ui=8080.

**Returns**: The number of relay IPs configured, or -1 on failure.

<a                                                    name="constants"><hr></a>

## Constants

PK_VERSION = "0.91.160601C"  
PK_STATUS_STARTUP = 10  
PK_STATUS_CONNECTING = 20  
PK_STATUS_UPDATING_DNS = 30  
PK_STATUS_FLYING = 40  
PK_STATUS_PROBLEMS = 50  
PK_STATUS_REJECTED = 60  
PK_STATUS_NO_NETWORK = 90  
PK_WITH_DEFAULTS = 0x8000  
PK_WITHOUT_DEFAULTS = 0x4000  
PK_WITH_SSL = 0x0001  
PK_WITH_IPV4 = 0x0002  
PK_WITH_IPV6 = 0x0004  
PK_WITH_SERVICE_FRONTENDS = 0x0008  
PK_WITHOUT_SERVICE_FRONTENDS = 0x0010  
PK_WITH_DYNAMIC_FE_LIST = 0x0020  
PK_WITH_FRONTEND_SNI = 0x0040  
PK_AS_FRONTEND_RELAY = 0x0100  
PK_LOG_TUNNEL_DATA = 0x000100  
PK_LOG_TUNNEL_HEADERS = 0x000200  
PK_LOG_TUNNEL_CONNS = 0x000400  
PK_LOG_BE_DATA = 0x001000  
PK_LOG_BE_HEADERS = 0x002000  
PK_LOG_BE_CONNS = 0x004000  
PK_LOG_MANAGER_ERROR = 0x010000  
PK_LOG_MANAGER_INFO = 0x020000  
PK_LOG_MANAGER_DEBUG = 0x040000  
PK_LOG_TRACE = 0x080000  
PK_LOG_LUA_DEBUG = 0x008000  
PK_LOG_LUA_INFO = 0x000800  
PK_LOG_ERROR = 0x100000  
PK_LOG_ERRORS = (PK_LOG_ERROR|PK_LOG_MANAGER_ERROR)  
PK_LOG_MANAGER = (PK_LOG_MANAGER_ERROR|PK_LOG_MANAGER_INFO)  
PK_LOG_CONNS = (PK_LOG_BE_CONNS|PK_LOG_TUNNEL_CONNS)  
PK_LOG_NORMAL = (PK_LOG_ERRORS|PK_LOG_CONNS|PK_LOG_MANAGER|PK_LOG_LUA_INFO)  
PK_LOG_DEBUG = (PK_LOG_NORMAL|PK_LOG_MANAGER_DEBUG|PK_LOG_LUA_DEBUG)  
PK_LOG_ALL = 0xffff00  