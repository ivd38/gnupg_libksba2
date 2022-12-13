
GNUPG libksba 1.6.1 overflow

<pre>
static gpg_error_t
parse_to_next_update (ksba_crl_t crl)
{
...
if ( !(ti.class == CLASS_UNIVERSAL && ti.tag == TYPE_SEQUENCE
         && ti.is_constructed) )
    return gpg_error (GPG_ERR_INV_CRL_OBJ);
  if (!tbs_ndef)
    {
      if (tbs_len < ti.nhdr)
        return gpg_error (GPG_ERR_BAD_BER);
      tbs_len -= ti.nhdr;
      if (!ti.ndef && tbs_len < ti.length)
        return gpg_error (GPG_ERR_BAD_BER);
      tbs_len -= ti.length;
    }
[1]  if (ti.nhdr + ti.length >= DIM(tmpbuf))
    return gpg_error (GPG_ERR_TOO_LARGE);
  memcpy (tmpbuf, ti.buf, ti.nhdr);
[2]  err = read_buffer (crl->reader, tmpbuf+ti.nhdr, ti.length);
</pre>

We control ti.length on line #1.

By setting it to -1 we can bypass this check.

As a result stack overflow happens on line #2


How to test:
<pre>
1. build libksba with asan
2. edit tests/t-crl-parser.c to open 1.der file
3. run tests/t-crl-parser

OR

$ dirmngr --load-crl 1.der
</pre>


Asan log attached.
