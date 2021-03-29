# ip地址格式规范

## ipv4

在CIDR表示法中，前缀显示为4个八位字节，就像传统的IPv4地址一样，后跟"/"斜杠和一个0～32之间的十进制值来描述有效位数。[^1]

> For example, the legacy "Class B" network 172.16.0.0, with an implied network mask of 255.255.0.0, is defined as the prefix 172.16.0.0/16, the "/16" indicating that the mask to extract the network portion of the prefix is a 32-bit value where the most significant 16 bits are ones and the least significant 16 bits are zeros.  
> Similarly, the legacy "Class C" network number 192.168.99.0 is defined as the prefix 192.168.99.0/24; the most significant 24 bits are ones and the least significant 8 bits are zeros.

## ipv6

There are three conventional forms for representing IPv6 addresses as text strings: [^2]

1. The preferred form is `x:x:x:x:x:x:x:x`, where the 'x's are the hexadecimal values of the eight 16-bit pieces of the address.
Examples:
   
   `FEDC:BA98:7654:3210:FEDC:BA98:7654:3210`
   
   `1080:0:0:0:8:800:200C:417A`

	Note that it is not necessary to write the leading zeros in an individual field, but there must be at least one numeral in every field (except for the case described in 2.).

2. Due to some methods of allocating certain styles of IPv6 addresses, it will be common for addresses to contain long strings of zero bits. In order to make writing addresses containing zero bits easier a special syntax is available to compress the zeros. The use of "::" indicates multiple groups of 16-bits of zeros. The "::" can only appear once in an address.  The "::" can also be used to compress the leading and/or trailing zeros in an address.
For example the following addresses:

   `1080:0:0:0:8:800:200C:417A`  a unicast address

   `FF01:0:0:0:0:0:0:101`        a multicast address

   `0:0:0:0:0:0:0:1`             the loopback address

   `0:0:0:0:0:0:0:0`             the unspecified addresses

	may be represented as:

   `1080::8:800:200C:417A`       a unicast address

   `FF01::101`                   a multicast address

   `::1`                         the loopback address

   `::`                          the unspecified addresses

3. An alternative form that is sometimes more convenient when dealing with a mixed environment of IPv4 and IPv6 nodes is `x:x:x:x:x:x:d.d.d.d`, where the 'x's are the hexadecimal values of the six high-order 16-bit pieces of the address, and the 'd's are the decimal values of the four low-order 8-bit pieces of the address (standard IPv4 representation).  Examples:

   `0:0:0:0:0:0:13.1.68.3`

   `0:0:0:0:0:FFFF:129.144.52.38`

	or in compressed form:

   `::13.1.68.3`

   `::FFFF:129.144.52.38`

IPv6地址前缀的文本表示类似于以CIDR表示法编写IPv4地址前缀的方式: IPv6地址/有效位数

# IPy模块的使用

安装Ipy模块

```shell
pip install IPy
```

IPy模块包含IP类，使用它可以处理绝大部分格式的IPv4或IPv6地址[^3]。

>It can detect about a dozen different ways of expressing IP addresses and networks, parse them and distinguish between IPv4 and IPv6 addresses:

通过version方法来区分出IPv4和IPv6

```python
>>> IP('10.0.0.0/8').version()
4
>>> IP('::1').version()
6
```
通过strNormal指定不同的wantprefixlen值控制输出

```python
>>> IP('10.0.0.0/32').strNormal()
'10.0.0.0'
>>> IP('10.0.0.0/24').strNormal()
'10.0.0.0/24'
>>> IP('10.0.0.0/24').strNormal(0)
'10.0.0.0'
>>> IP('10.0.0.0/24').strNormal(1)
'10.0.0.0/24'
>>> IP('10.0.0.0/24').strNormal(2)
'10.0.0.0/255.255.255.0'
>>> IP('10.0.0.0/24').strNormal(3)
'10.0.0.0-10.0.0.255'
>>> ip = IP('10.0.0.0')
>>> print(ip)
10.0.0.0
>>> ip.NoPrefixForSingleIp = None
>>> print(ip)
10.0.0.0/32
>>> ip.WantPrefixLen = 3
>>> print(ip)
10.0.0.0-10.0.0.0
```

# 实现代码

```python
from IPy import IP

def is_ip(ip_str):
	try:
		ip = IP(ip_str)
		return True
	except Exception as e:
		print("The address is illegal.")
		return False

ip_str = input("Please enter the IP address:\n ")
if is_ip(ip_str):
	ip = IP(ip_str)
	version = ip.version()
	print("The address is IPv", version, sep='')
	if ip.len() > 1:
		print("Available address segment is:", ip.strNormal(3))
		print("The number of address is:", ip.len())
	else:
		print("The binary address is:", ip.strBin())

```

# 测试结果

```
testcase 1
Please enter the IP address:
 0.0.0.0
The address is IPv4
The binary address is: 00000000000000000000000000000000
---------------------------------
testcase 2
Please enter the IP address:
 255.255.255.255
The address is IPv4
The binary address is: 11111111111111111111111111111111
---------------------------------
testcase 3
Please enter the IP address:
 256.0.1.1
The address is illegal.
---------------------------------
testcase 4
Please enter the IP address:
 192.168.0.0/24
The address is IPv4
Available address segment is: 192.168.0.0-192.168.0.255
The number of address is: 256
---------------------------------
testcase 5
Please enter the IP address:
 192.168.1.1/24
The address is illegal.
---------------------------------
testcase 6
Please enter the IP address:
 ::1/128
The address is IPv6
The binary address is: 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001
---------------------------------
testcase 7
Please enter the IP address:
 FEDC:BA98:7654:3210:FEDC:BA98:7654:3210
The address is IPv6
The binary address is: 11111110110111001011101010011000011101100101010000110010000100001111111011011100101110101001100001110110010101000011001000010000
---------------------------------
testcase 8
Please enter the IP address:
 1080::8:800:200C:417A/24
The address is illegal.
---------------------------------
testcase 9
Please enter the IP address:
 FF01::101
The address is IPv6
The binary address is: 11111111000000010000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000100000001
---------------------------------
testcase 10
Please enter the IP address:
 ::
The address is IPv6
The binary address is: 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
---------------------------------
testcase 11
Please enter the IP address:
 ::/64
The address is IPv6
Available address segment is: 0:0:0:0:0:0:0:0-0000:0000:0000:0000:ffff:ffff:ffff:ffff
The number of address is: 18446744073709551616
---------------------------------
```

# IPy源码分析

> IPy - class and tools for handling of IPv4 and IPv6 addresses and networks.

源码版本为1.01 `__version__ = '1.01'` [^4]

## ipversion

```python
if isinstance(data, INT_TYPES):
            self.ip = int(data)
            if ipversion == 0:
                if self.ip <= MAX_IPV4_ADDRESS:
                    ipversion = 4
                else:
                    ipversion = 6
            if ipversion == 4:
                if self.ip > MAX_IPV4_ADDRESS:
                    raise ValueError("IPv4 Address can't be larger than %x: %x" % (MAX_IPV4_ADDRESS, self.ip))
                prefixlen = 32
            elif ipversion == 6:
                if self.ip > MAX_IPV6_ADDRESS:
                    raise ValueError("IPv6 Address can't be larger than %x: %x" % (MAX_IPV6_ADDRESS, self.ip))
                prefixlen = 128
            else:
                raise ValueError("only IPv4 and IPv6 supported")
```

根据处理后的ip长度判断类型，其中`MAX_IPV4_ADDRESS = 0xffffffff, MAX_IPV6_ADDRESS = 0xffffffffffffffffffffffffffffffff`

## strbin

```python
bits = _ipVersionToLen(self._ipversion)
if self.WantPrefixLen == None and wantprefixlen == None:
	wantprefixlen = 0
ret = _intToBin(self.ip)
return  '0' * (bits - len(ret)) + ret + self._printPrefix(wantprefixlen)
```

## strnormal

```python
if self.WantPrefixLen == None and wantprefixlen == None:
	wantprefixlen = 1

if self._ipversion == 4:
	ret = self.strFullsize(0) # Return a string representation in the non-mangled format.
elif self._ipversion == 6:
	ret = ':'.join(["%x" % x for x in [int(x, 16) for x in self.strFullsize(0).split(':')]])
else:
	raise ValueError("only IPv4 and IPv6 supported")
```

## len

```python
bits = _ipVersionToLen(self._ipversion) # Return number of bits in address for a certain IP version.(32 or 128)
locallen = bits - self._prefixlen
return 2 ** locallen
```

范围IP分解

```python
if isinstance(key, slice):
	return [IP(IPint.__getitem__(self, x), ipversion=self._ipversion) for x in xrange(*key.indices(len(self)))]
	return IP(IPint.__getitem__(self, key), ipversion=self._ipversion)
```

## getIPv4Map

```python
if self._ipversion != 6:
	return None
if (self.ip >> 32) != 0xffff:
	return None
ipv4 = self.ip & MAX_IPV4_ADDRESS
if self._prefixlen != 128:
	ipv4 = '%s/%s' % (ipv4, 32-(128-self._prefixlen))
return IP(ipv4, ipversion=4)
```

## IP输入格式检测

解析字符串并返回相应的整数型IP地址

```python
def parseAddress(ipstr, ipversion=0):
    try:
        hexval = int(ipstr, 16)
    except ValueError:
        hexval = None
    try:
        intval = int(ipstr, 10)
    except ValueError:
        intval = None

    if ipstr.startswith('0x') and hexval is not None:
        if hexval > MAX_IPV6_ADDRESS:
            raise ValueError("IP Address can't be larger than %x: %x" % (MAX_IPV6_ADDRESS, hexval))
        if hexval <= MAX_IPV4_ADDRESS:
            return (hexval, 4)
        else:
            return (hexval, 6)

    if ipstr.find(':') != -1:
        return (_parseAddressIPv6(ipstr), 6)

    elif len(ipstr) == 32 and hexval is not None:
        # assume IPv6 in pure hexadecimal notation
        return (hexval, 6)

    elif ipstr.find('.') != -1 or (intval is not None and intval < 256 and ipversion != 6):
        # assume IPv4  ('127' gets interpreted as '127.0.0.0')
        bytes = ipstr.split('.')
        if len(bytes) > 4:
            raise ValueError("IPv4 Address with more than 4 bytes")
        bytes += ['0'] * (4 - len(bytes))
        bytes = [int(x) for x in bytes]
        for x in bytes:
            if x > 255 or x < 0:
                raise ValueError("%r: single byte must be 0 <= byte < 256" % (ipstr))
        return ((bytes[0] << 24) + (bytes[1] << 16) + (bytes[2] << 8) + bytes[3], 4)

    elif intval is not None:
        # we try to interprete it as a decimal digit -
        # this ony works for numbers > 255 ... others
        # will be interpreted as IPv4 first byte
        if intval > MAX_IPV6_ADDRESS:
            raise ValueError("IP Address can't be larger than %x: %x" % (MAX_IPV6_ADDRESS, intval))
        if intval <= MAX_IPV4_ADDRESS and ipversion != 6:
            return (intval, 4)
        else:
            return (intval, 6)

    raise ValueError("IP Address format was invalid: %s" % ipstr)
```

分解IPv6地址

```python
def _parseAddressIPv6(ipstr):

    items = []
    index = 0
    fill_pos = None
    while index < len(ipstr):
        text = ipstr[index:]
        if text.startswith("::"):
            if fill_pos is not None:
                # Invalid IPv6, eg. '1::2::'
                raise ValueError("%r: Invalid IPv6 address: more than one '::'" % ipstr)
            fill_pos = len(items)
            index += 2
            continue
        pos = text.find(':')
        if pos == 0:
            # Invalid IPv6, eg. '1::2:'
            raise ValueError("%r: Invalid IPv6 address" % ipstr)
        if pos != -1:
            items.append(text[:pos])
            if text[pos:pos+2] == "::":
                index += pos
            else:
                index += pos+1

            if index == len(ipstr):
                # Invalid IPv6, eg. '1::2:'
                raise ValueError("%r: Invalid IPv6 address" % ipstr)
        else:
            items.append(text)
            break

    if items and '.' in items[-1]:
        # IPv6 ending with IPv4 like '::ffff:192.168.0.1'
        if (fill_pos is not None) and not (fill_pos <= len(items)-1):
            # Invalid IPv6: 'ffff:192.168.0.1::'
            raise ValueError("%r: Invalid IPv6 address: '::' after IPv4" % ipstr)
        value = parseAddress(items[-1])[0]
        items = items[:-1] + ["%04x" % (value >> 16), "%04x" % (value & 0xffff)]

    # Expand fill_pos to fill with '0'
    # ['1','2'] with fill_pos=1 => ['1', '0', '0', '0', '0', '0', '0', '2']
    if fill_pos is not None:
        diff = 8 - len(items)
        if diff <= 0:
            raise ValueError("%r: Invalid IPv6 address: '::' is not needed" % ipstr)
        items = items[:fill_pos] + ['0']*diff + items[fill_pos:]

    # Here we have a list of 8 strings
    if len(items) != 8:
        # Invalid IPv6, eg. '1:2:3'
        raise ValueError("%r: Invalid IPv6 address: should have 8 hextets" % ipstr)

    # Convert strings to long integer
    value = 0
    index = 0
    for item in items:
        try:
            item = int(item, 16)
            error = not(0 <= item <= 0xffff)
        except ValueError:
            error = True
        if error:
            raise ValueError("%r: Invalid IPv6 address: invalid hexlet %r" % (ipstr, item))
        value = (value << 16) + item
        index += 1
    return value
```

通过分割移位的方式转换输入为IP

# 参考链接

[^1]: RFC 791; RFC 4632
[^2]: RFC 2373
[^3]: https://pypi.org/project/IPy/#description
[^4]: https://github.com/autocracy/python-ipy