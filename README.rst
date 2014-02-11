System Administration with Ansible
==================================

This repository contains slides and examples for a presentation on using `Ansible
<http://www.ansible.com>`_ for automating system administration.

The example in `ansible/` uses roles to configure a remote host for serving a
static blog built using `Pelican <http://docs.getpelican.com/>`_ and comments
for that static blog using `Isso <http://posativ.org/isso/>`_.

The playbook was designed for `Fedora <https://fedoraproject.org>`_ 20 and may
not work with other distributions out of the box.

Slides are in the `presentation/` directory and were designed for use with
`rst2html5 <https://github.com/marianoguerra/rst2html5>`_ and `Deck.js <http://imakewebthings.com/deck.js/>`_.


License and Credit
------------------

Unless otherwise noted, the contents are available under a `cc-by-sa 4.0
international licence <http://creativecommons.org/licenses/by-sa/4.0/>`_

The foundation-default-colors theme for pelican is courtesy of `Kenton Hamaluik <https://github.com/FuzzyWuzzie>`_.


Other Requirements
------------------

On the local host, you'll need the following to run the example playbook:

  * Ansible >= 1.4

  * Pelican >= 3.3


