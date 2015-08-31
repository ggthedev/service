service: Write daemons in Go
----------------------------

[![GoDoc](https://godoc.org/github.com/hlandau/service?status.svg)](https://godoc.org/github.com/hlandau/service)

This package enables you to easily write services in Go such that the following concerns are taken care of automatically:

  - Daemonization
  - Fork emulation (not recommended, though)
  - PID file creation
  - Privilege dropping
  - Chrooting
  - Status notification (supports setproctitle and systemd notify protocol; this support is Go-native and does not introduce any dependency on any systemd library)
  - Operation as a Windows service
  - Orderly shutdown

Here's a usage example:

    import "github.com/hlandau/service"

    func main() {
      service.Main(&service.Info{
          Title:       "Foobar Web Server",
          Name:        "foobar",
          Description: "Foobar Web Server is the greatest webserver ever.",

          RunFunc: func(smgr service.Manager) error {
              // Start up your service.
              // ...

              // Once initialization requiring root is done, call this.
              err := smgr.DropPrivileges()
              if err != nil {
                  return err
              }

              // When it is ready to serve requests, call this.
              // You must call DropPrivileges first.
              smgr.SetStarted()

              // Optionally set a status
              smgr.SetStatus("foobar: running ok")

              // Wait until stop is requested.
              <-smgr.StopChan()

              // Do any necessary teardown.
              // ...

              // Done.
              return nil
          },
      })
    }

Flags
=====

The following flags are automatically registered via the "flag" package:

    -chroot=path              (*nix only) chroot to a directory (must set UID, GID) ("/" disables)
    -daemon=0|1               (*nix only) run as daemon? (doesn't fork)
    -dropprivs=0|1            (*nix only) drop privileges?
    -fork=0|1                 (*nix only) fork?
    -uid=username             (*nix only) UID or username to setuid to
    -gid=groupname            (*nix only) GID or group name to setgid to
    -pidfile=path             (*nix only) Path of PID file to write and lock (default: no PID file)
    -cpuprofile=path          Write CPU profile to file
    -debugserveraddr=ip:port  Bind the net/http DefaultServeMux to the given address
                              (expvars, pprof handlers will be registered; intended for debug use only;
                               set UsesDefaultHTTP in the Info type to disable the presence of this flag)
    -service=start|stop|install|remove  (Windows only) Service control.

Licence
=======

    ISC License

    Permission to use, copy, modify, and distribute this software for any
    purpose with or without fee is hereby granted, provided that the above
    copyright notice and this permission notice appear in all copies.

    THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
    WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
    MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
    ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
    WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
    ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
    OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

