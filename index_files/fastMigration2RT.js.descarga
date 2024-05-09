// All code in this file will run on fast migration sites
(function () {
    'use strict';

    var methods = {
        onDocReady: _onDocReady,
    };

    ready(methods.onDocReady);

    // VANILLA DOCUMENT READY (NO JQUERY)
    // supports IE8 and above, not tested for old browsers
    function ready(fn) {
        if (document.readyState != 'loading') {
            fn();
        } else if (document.addEventListener) {
            document.addEventListener('DOMContentLoaded', fn);
        } else {
            document.attachEvent('onreadystatechange', function () {
                if (document.readyState != 'loading') fn();
            });
        }
    }

    function _onDocReady() {
        if (
            document.cookie &&
            document.cookie.indexOf('persistentCookieExists=true') > -1
        ) {
            $('#cookieWrapper').hide();
        }
    }

    document.addEventListener('DOMContentLoaded', function (event) {
        formModifier();
    });

    function formModifier() {
        var forms = document.querySelectorAll('form:not([acction^="http"])');

        for (var i = 0; i < forms.length; i++) {
            var form = forms[i];

            var actionAttr = form.getAttribute('action');
            if (actionAttr && actionAttr.indexOf('paypal') > 0) {
                continue;
            }

            if (actionAttr && actionAttr.endsWith('.aspx')) {
                continue;
            }

            if (form.getAttribute('dmnochange')) {
                continue;
            }

            form.setAttribute('method', 'POST');
            form.setAttribute(
                'action',
                '/_dm/s/rt/actions/fm/form/action?a=' + siteDetails.SiteAlias
            );
            form.setAttribute('dm_dont_rewrite_url', 'true');
            var allInputs = form.querySelectorAll('input, textarea,select');
            for (var k = 0; k < allInputs.length; k++) {
                var input = allInputs[k];
                var title = input.getAttribute('title');
                var titleEncoded = encodeURI(title);
                var inputName = input.getAttribute('name');
                var formName = form.getAttribute('name');
                if (inputName && formName) {
                    input.setAttribute(
                        'name',
                        inputName
                            .replace(formName + '_', '')
                            .replace(formName, '')
                    );
                }
                if (title) {
                    title = title.replace(':', '');
                    input.setAttribute('name', titleEncoded);
                }

                if (input.type === 'select-one') {
                    changeValueToTextContent(input);
                }

                if (input.type === 'file') {
                    if (!titleEncoded) {
                        titleEncoded = 'Attachment';
                    }
                    input.setAttribute('name', titleEncoded + '_FILE');
                }
            }
            var subjectMail = document.createElement('input');
            subjectMail.setAttribute('type', 'hidden');
            subjectMail.setAttribute('name', 'dm-form-m-ignore-subject');
            subjectMail.setAttribute(
                'value',
                'You recevied a new mail from one' + ' of your sites'
            );
            form.appendChild(subjectMail);

            var mailField = document.createElement('input');
            mailField.setAttribute('type', 'hidden');
            mailField.setAttribute('name', 'dm-form-m-ignore-sendto');
            mailField.setAttribute(
                'value',
                window.fsBodyEnd && window.fsBodyEnd.sendToEmail
            );
            form.appendChild(mailField);

            var encryptedMailField = document.createElement('input');
            encryptedMailField.setAttribute('type', 'hidden');
            encryptedMailField.setAttribute(
                'name',
                'dm-form-m-ignore-sendto-enc'
            );
            encryptedMailField.setAttribute(
                'value',
                window.fsBodyEnd && window.fsBodyEnd.sendToEmailEncrypted
            );
            form.appendChild(encryptedMailField);

            var allHiddenElement = form.querySelectorAll('[type=hidden]');
            var ignoredAsArray = '';
            for (var j = 0; j < allHiddenElement.length; j++) {
                var element = allHiddenElement[j];
                ignoredAsArray += element.getAttribute('name');
                if (j < allHiddenElement.length - 1) {
                    ignoredAsArray += ',';
                }
            }

            var ignoreIndicatorElement = document.createElement('input');
            ignoreIndicatorElement.setAttribute('type', 'hidden');
            ignoreIndicatorElement.setAttribute('name', 'dm-form-m-ignore');
            ignoreIndicatorElement.setAttribute('value', ignoredAsArray);
            form.appendChild(ignoreIndicatorElement);
            overrideSubmitForm(form);
        }
    }

    function changeValueToTextContent(input) {
        var options = input.options;
        for (var i = 0; i < options.length; i++) {
            options[i].value = options[i].textContent;
        }
    }

    function getStyle(element, attr) {
        return document.defaultView
            .getComputedStyle(element, null)
            .getPropertyValue(attr);
    }

    function overrideSubmitForm(form) {
        var formBtn = document.querySelector("form button[type='submit']");
        if (formBtn) {
            formBtn.removeAttribute('onclick');
        }
        var successDiv = document.createElement('div');
        form.onsubmit = function (event) {
            event.preventDefault();
            var url = form.getAttribute('action');

            var data = new FormData(form);
            data.delete('cc-id');
            data.delete('cc-ntotal');
            var req = new XMLHttpRequest();
            req.open('POST', url, true);

            req.onreadystatechange = function () {
                //Call a function when the state changes.
                if (
                    req.readyState == XMLHttpRequest.DONE &&
                    req.status == 200
                ) {
                    var textToShow = req.responseText;
                    if (req.responseText.startsWith('ERROR_PREFIX')) {
                        textToShow = textToShow.replace('ERROR_PREFIX', '');
                    }
                    var submitBtn = form.querySelector('input[type=submit]');
                    successDiv.innerHTML = textToShow;
                    form.style.display = 'none';
                    if (getStyle(form, 'position') == 'absolute') {
                        successDiv.style.top = getStyle(form, 'top');
                        successDiv.style.left = getStyle(form, 'left');
                        successDiv.style.position = getStyle(form, 'position');
                    }
                    form.parentNode.insertBefore(successDiv, form.nextSibling);
                }
            };
            req.send(data);

            return false;
        };
    }

    //
    window.emptyFunction = function () {};
    var allRelativeAnchors = document.querySelectorAll('a:not([href^="http"])');
    for (var i = 0, len = allRelativeAnchors.length; i < len; i++) {
        var anchor = allRelativeAnchors[i];
        if (
            anchor.getAttribute('href') &&
            anchor.getAttribute('href') !== '#'
        ) {
            var onclick = anchor.getAttribute('onclick');
            if (onclick && window.isPreview) {
                onclick = onclick.replace(
                    'window.open',
                    'window.emptyFunction'
                );
                if (onclick.indexOf('scroll') < 0) {
                    onclick = onclick.replace('return false;', '');
                }
                anchor.setAttribute('onclick', onclick);
            } else {
                anchor.setAttribute('onclick', '');
            }
        }
        var homePage = 'index.html';
        var hrefAttr = anchor.getAttribute('href');

        const dm_device = getParameterByName('dm_device');
        if (dm_device && hrefAttr && hrefAttr.indexOf('dm_device') < 0) {
            anchor.href = hrefAttr += '?dm_device=' + dm_device;
        }

        if (location.pathname !== '/') {
            continue;
        }

        if (
            hrefAttr &&
            hrefAttr.startsWith(homePage) &&
            hrefAttr !== homePage
        ) {
            anchor.href = hrefAttr.replace(homePage, '');
        }
    }

    function getParameterByName(name, url) {
        if (!url) url = window.location.href;
        name = name.replace(/[\[\]]/g, '\\$&');
        var regex = new RegExp('[?&]' + name + '(=([^&#]*)|&|#|$)'),
            results = regex.exec(url);
        if (!results) return null;
        if (!results[2]) return '';
        return decodeURIComponent(results[2].replace(/\+/g, ' '));
    }
})();
