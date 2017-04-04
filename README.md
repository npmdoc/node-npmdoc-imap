# api documentation for  [imap (v0.8.19)](https://github.com/mscdex/node-imap#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-imap.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-imap) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-imap.svg)](https://travis-ci.org/npmdoc/node-npmdoc-imap)
#### An IMAP module for node.js that makes communicating with IMAP servers easy

[![NPM](https://nodei.co/npm/imap.png?downloads=true)](https://www.npmjs.com/package/imap)

[![apidoc](https://npmdoc.github.io/node-npmdoc-imap/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-imap_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-imap/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-imap/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-imap/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Brian White",
        "email": "mscdex@mscdex.net"
    },
    "bugs": {
        "url": "https://github.com/mscdex/node-imap/issues"
    },
    "dependencies": {
        "readable-stream": "1.1.x",
        "utf7": ">=1.0.2"
    },
    "description": "An IMAP module for node.js that makes communicating with IMAP servers easy",
    "devDependencies": {},
    "directories": {},
    "dist": {
        "shasum": "3678873934ab09cea6ba48741f284da2af59d8d5",
        "tarball": "https://registry.npmjs.org/imap/-/imap-0.8.19.tgz"
    },
    "engines": {
        "node": ">=0.8.0"
    },
    "homepage": "https://github.com/mscdex/node-imap#readme",
    "keywords": [
        "imap",
        "mail",
        "email",
        "reader",
        "client"
    ],
    "licenses": [
        {
            "type": "MIT",
            "url": "http://github.com/mscdex/node-imap/raw/master/LICENSE"
        }
    ],
    "main": "./lib/Connection",
    "maintainers": [
        {
            "name": "mscdex",
            "email": "mscdex@mscdex.net"
        }
    ],
    "name": "imap",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/mscdex/node-imap.git"
    },
    "scripts": {
        "test": "node test/test.js"
    },
    "version": "0.8.19"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module imap](#apidoc.module.imap)
1.  [function <span class="apidocSignatureSpan">imap.</span>parseHeader (str, noDecode)](#apidoc.element.imap.parseHeader)
1.  [function <span class="apidocSignatureSpan">imap.</span>super_ ()](#apidoc.element.imap.super_)
1.  object <span class="apidocSignatureSpan">imap.</span>Parser

#### [module imap.Parser](#apidoc.module.imap.Parser)
1.  [function <span class="apidocSignatureSpan">imap.</span>Parser (stream, debug)](#apidoc.element.imap.Parser.Parser)
1.  [function <span class="apidocSignatureSpan">imap.Parser.</span>parseBodyStructure (cur, literals, prefix, partID)](#apidoc.element.imap.Parser.parseBodyStructure)
1.  [function <span class="apidocSignatureSpan">imap.Parser.</span>parseEnvelopeAddresses (list)](#apidoc.element.imap.Parser.parseEnvelopeAddresses)
1.  [function <span class="apidocSignatureSpan">imap.Parser.</span>parseExpr (o, literals, result, start, useBrackets)](#apidoc.element.imap.Parser.parseExpr)
1.  [function <span class="apidocSignatureSpan">imap.Parser.</span>parseHeader (str, noDecode)](#apidoc.element.imap.Parser.parseHeader)



# <a name="apidoc.module.imap"></a>[module imap](#apidoc.module.imap)

#### <a name="apidoc.element.imap.parseHeader"></a>[function <span class="apidocSignatureSpan">imap.</span>parseHeader (str, noDecode)](#apidoc.element.imap.parseHeader)
- description and source-code
```javascript
function parseHeader(str, noDecode) {
  var lines = str.split(RE_CRLF),
      len = lines.length,
      header = {},
      state = {
        buffer: undefined,
        encoding: undefined,
        consecutive: false,
        replaces: undefined,
        curReplace: undefined,
        remainder: undefined
      },
      m, h, i, val;

  for (i = 0; i < len; ++i) {
    if (lines[i].length === 0)
      break; // empty line separates message's header and body
    if (lines[i][0] === '\t' || lines[i][0] === ' ') {
      if (!Array.isArray(header[h]))
        continue; // ignore invalid first line
      // folded header content
      val = lines[i];
      if (!noDecode) {
        if (RE_ENCWORD_END.test(lines[i - 1])
            && RE_ENCWORD_BEGIN.test(val)) {
          // RFC2047 says to *ignore* leading whitespace in folded header values
          // for adjacent encoded-words ...
          val = val.substring(1);
        }
      }
      header[h][header[h].length - 1] += val;
    } else {
      m = RE_HDR.exec(lines[i]);
      if (m) {
        h = m[1].toLowerCase().trim();
        if (m[2]) {
          if (header[h] === undefined)
            header[h] = [m[2]];
          else
            header[h].push(m[2]);
        } else
          header[h] = [''];
      } else
        break;
    }
  }
  if (!noDecode) {
    var hvs;
    for (h in header) {
      hvs = header[h];
      for (i = 0, len = header[h].length; i < len; ++i)
        hvs[i] = decodeWords(hvs[i], state);
    }
  }

  return header;
}
```
- example usage
```shell
...
var prefix = '(#' + seqno + ') ';
msg.on('body', function(stream, info) {
  var buffer = '';
  stream.on('data', function(chunk) {
    buffer += chunk.toString('utf8');
  });
  stream.once('end', function() {
    console.log(prefix + 'Parsed header: %s', inspect(Imap.parseHeader(buffer)));
  });
});
msg.once('attributes', function(attrs) {
  console.log(prefix + 'Attributes: %s', inspect(attrs, false, 8));
});
msg.once('end', function() {
  console.log(prefix + 'Finished');
...
```

#### <a name="apidoc.element.imap.super_"></a>[function <span class="apidocSignatureSpan">imap.</span>super_ ()](#apidoc.element.imap.super_)
- description and source-code
```javascript
function EventEmitter() {
  EventEmitter.init.call(this);
}
```
- example usage
```shell
n/a
```



# <a name="apidoc.module.imap.Parser"></a>[module imap.Parser](#apidoc.module.imap.Parser)

#### <a name="apidoc.element.imap.Parser.Parser"></a>[function <span class="apidocSignatureSpan">imap.</span>Parser (stream, debug)](#apidoc.element.imap.Parser.Parser)
- description and source-code
```javascript
function Parser(stream, debug) {
  if (!(this instanceof Parser))
    return new Parser(stream, debug);

  EventEmitter.call(this);

  this._stream = undefined;
  this._body = undefined;
  this._literallen = 0;
  this._literals = [];
  this._buffer = '';
  this._ignoreReadable = false;
  this.debug = debug;

  var self = this;
  this._cbReadable = function() {
    if (self._ignoreReadable)
      return;
    if (self._literallen > 0 && !self._body)
      self._tryread(self._literallen);
    else
      self._tryread();
  };

  this.setStream(stream);

  process.nextTick(this._cbReadable);
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.imap.Parser.parseBodyStructure"></a>[function <span class="apidocSignatureSpan">imap.Parser.</span>parseBodyStructure (cur, literals, prefix, partID)](#apidoc.element.imap.Parser.parseBodyStructure)
- description and source-code
```javascript
function parseBodyStructure(cur, literals, prefix, partID) {
  var ret = [], i, len;
  if (prefix === undefined) {
    var result = (Array.isArray(cur) ? cur : parseExpr(cur, literals));
    if (result.length)
      ret = parseBodyStructure(result, literals, '', 1);
  } else {
    var part, partLen = cur.length, next;
    if (Array.isArray(cur[0])) { // multipart
      next = -1;
      while (Array.isArray(cur[++next])) {
        ret.push(parseBodyStructure(cur[next],
                                    literals,
                                    prefix + (prefix !== '' ? '.' : '')
                                           + (partID++).toString(), 1));
      }
      part = { type: cur[next++].toLowerCase() };
      if (partLen > next) {
        if (Array.isArray(cur[next])) {
          part.params = {};
          for (i = 0, len = cur[next].length; i < len; i += 2)
            part.params[cur[next][i].toLowerCase()] = cur[next][i + 1];
        } else
          part.params = cur[next];
        ++next;
      }
    } else { // single part
      next = 7;
      if (typeof cur[1] === 'string') {
        part = {
          // the path identifier for this part, useful for fetching specific
          // parts of a message
          partID: (prefix !== '' ? prefix : '1'),

          // required fields as per RFC 3501 -- null or otherwise
          type: cur[0].toLowerCase(), subtype: cur[1].toLowerCase(),
          params: null, id: cur[3], description: cur[4], encoding: cur[5],
          size: cur[6]
        };
      } else {
        // type information for malformed multipart body
        part = { type: cur[0] ? cur[0].toLowerCase() : null, params: null };
        cur.splice(1, 0, null);
        ++partLen;
        next = 2;
      }
      if (Array.isArray(cur[2])) {
        part.params = {};
        for (i = 0, len = cur[2].length; i < len; i += 2)
          part.params[cur[2][i].toLowerCase()] = cur[2][i + 1];
        if (cur[1] === null)
          ++next;
      }
      if (part.type === 'message' && part.subtype === 'rfc822') {
        // envelope
        if (partLen > next && Array.isArray(cur[next]))
          part.envelope = parseFetchEnvelope(cur[next]);
        else
          part.envelope = null;
        ++next;

        // body
        if (partLen > next && Array.isArray(cur[next]))
          part.body = parseBodyStructure(cur[next], literals, prefix, 1);
        else
          part.body = null;
        ++next;
      }
      if ((part.type === 'text'
           || (part.type === 'message' && part.subtype === 'rfc822'))
          && partLen > next)
        part.lines = cur[next++];
      if (typeof cur[1] === 'string' && partLen > next)
        part.md5 = cur[next++];
    }
    // add any extra fields that may or may not be omitted entirely
    parseStructExtra(part, partLen, cur, next);
    ret.unshift(part);
  }
  return ret;
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.imap.Parser.parseEnvelopeAddresses"></a>[function <span class="apidocSignatureSpan">imap.Parser.</span>parseEnvelopeAddresses (list)](#apidoc.element.imap.Parser.parseEnvelopeAddresses)
- description and source-code
```javascript
function parseEnvelopeAddresses(list) {
  var addresses = null;
  if (Array.isArray(list)) {
    addresses = [];
    var inGroup = false, curGroup;
    for (var i = 0, len = list.length, addr; i < len; ++i) {
      addr = list[i];
      if (addr[2] === null) { // end of group addresses
        inGroup = false;
        if (curGroup) {
          addresses.push(curGroup);
          curGroup = undefined;
        }
      } else if (addr[3] === null) { // start of group addresses
        inGroup = true;
        curGroup = {
          group: addr[2],
          addresses: []
        };
      } else { // regular user address
        var info = {
          name: decodeWords(addr[0]),
          mailbox: addr[2],
          host: addr[3]
        };
        if (inGroup)
          curGroup.addresses.push(info);
        else if (!inGroup)
          addresses.push(info);
      }
      list[i] = addr;
    }
    if (inGroup) {
      // no end of group found, assume implicit end
      addresses.push(curGroup);
    }
  }
  return addresses;
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.imap.Parser.parseExpr"></a>[function <span class="apidocSignatureSpan">imap.Parser.</span>parseExpr (o, literals, result, start, useBrackets)](#apidoc.element.imap.Parser.parseExpr)
- description and source-code
```javascript
function parseExpr(o, literals, result, start, useBrackets) {
  start = start || 0;
  var inQuote = false,
      lastPos = start - 1,
      isTop = false,
      isBody = false,
      escaping = false,
      val;

  if (useBrackets === undefined)
    useBrackets = true;
  if (!result)
    result = [];
  if (typeof o === 'string') {
    o = { str: o };
    isTop = true;
  }
  for (var i = start, len = o.str.length; i < len; ++i) {
    if (!inQuote) {
      if (isBody) {
        if (o.str[i] === ']') {
          val = convStr(o.str.substring(lastPos + 1, i + 1), literals);
          result.push(val);
          lastPos = i;
          isBody = false;
        }
      } else if (o.str[i] === '"')
        inQuote = true;
      else if (o.str[i] === ' '
               || o.str[i] === ')'
               || (useBrackets && o.str[i] === ']')) {
        if (i - (lastPos + 1) > 0) {
          val = convStr(o.str.substring(lastPos + 1, i), literals);
          result.push(val);
        }
        if ((o.str[i] === ')' || (useBrackets && o.str[i] === ']')) && !isTop)
          return i;
        lastPos = i;
      } else if ((o.str[i] === '(' || (useBrackets && o.str[i] === '['))) {
        if (o.str[i] === '['
            && i - 4 >= start
            && o.str.substring(i - 4, i).toUpperCase() === 'BODY') {
          isBody = true;
          lastPos = i - 5;
        } else {
          var innerResult = [];
          i = parseExpr(o, literals, innerResult, i + 1, useBrackets);
          lastPos = i;
          result.push(innerResult);
        }
      }
    } else if (o.str[i] === '\\')
      escaping = !escaping;
    else if (o.str[i] === '"') {
      if (!escaping)
        inQuote = false;
      escaping = false;
    }
    if (i + 1 === len && len - (lastPos + 1) > 0)
      result.push(convStr(o.str.substring(lastPos + 1), literals));
  }
  return (isTop ? result : start);
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.imap.Parser.parseHeader"></a>[function <span class="apidocSignatureSpan">imap.Parser.</span>parseHeader (str, noDecode)](#apidoc.element.imap.Parser.parseHeader)
- description and source-code
```javascript
function parseHeader(str, noDecode) {
  var lines = str.split(RE_CRLF),
      len = lines.length,
      header = {},
      state = {
        buffer: undefined,
        encoding: undefined,
        consecutive: false,
        replaces: undefined,
        curReplace: undefined,
        remainder: undefined
      },
      m, h, i, val;

  for (i = 0; i < len; ++i) {
    if (lines[i].length === 0)
      break; // empty line separates message's header and body
    if (lines[i][0] === '\t' || lines[i][0] === ' ') {
      if (!Array.isArray(header[h]))
        continue; // ignore invalid first line
      // folded header content
      val = lines[i];
      if (!noDecode) {
        if (RE_ENCWORD_END.test(lines[i - 1])
            && RE_ENCWORD_BEGIN.test(val)) {
          // RFC2047 says to *ignore* leading whitespace in folded header values
          // for adjacent encoded-words ...
          val = val.substring(1);
        }
      }
      header[h][header[h].length - 1] += val;
    } else {
      m = RE_HDR.exec(lines[i]);
      if (m) {
        h = m[1].toLowerCase().trim();
        if (m[2]) {
          if (header[h] === undefined)
            header[h] = [m[2]];
          else
            header[h].push(m[2]);
        } else
          header[h] = [''];
      } else
        break;
    }
  }
  if (!noDecode) {
    var hvs;
    for (h in header) {
      hvs = header[h];
      for (i = 0, len = header[h].length; i < len; ++i)
        hvs[i] = decodeWords(hvs[i], state);
    }
  }

  return header;
}
```
- example usage
```shell
...
var prefix = '(#' + seqno + ') ';
msg.on('body', function(stream, info) {
  var buffer = '';
  stream.on('data', function(chunk) {
    buffer += chunk.toString('utf8');
  });
  stream.once('end', function() {
    console.log(prefix + 'Parsed header: %s', inspect(Imap.parseHeader(buffer)));
  });
});
msg.once('attributes', function(attrs) {
  console.log(prefix + 'Attributes: %s', inspect(attrs, false, 8));
});
msg.once('end', function() {
  console.log(prefix + 'Finished');
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
