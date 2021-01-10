``test-helpers`` makes modular testing easier
=============================================

``test-helpers`` provides an abstraction which library authors can use to
simplify testing of end-user code.

For example, consider a simple messaging library which has the interface:

.. code:: python

    class MessagingLibrary:
        def send_message(self, topic, value):
            ...

Users of this library will likely want to be able to write unit tests which
mock the library, and make assertions about messages which are sent:

.. code:: python

    class TestUserCode(TestCase):
        messages = MessagingLibraryHelper()

        def test_widget_update(self):
            update_widget(id=42)
            self.messages.assert_message_sent("widget-updates", {"id": 42})

More specifically, ``test-helpers`` makes it possible to create modular,
composeable, classes to help with testing:

.. code:: python

    class MessagingLibraryHelper(TestHelper):
        mock_send_message = MockHelper("MessagingLibrary.send_message")

        def setup(self):
            self.sent_messages = []
            self.mock_send_message.set_value(self._mock_send_message)

        def _mock_send_message(self, topic, value):
            self.sent_messages.append((topic, value))

        def assert_message_sent(self, topic, value):
            for msg_topic, msg_value in self.sent_messages:
                if msg_topic == topic and msg_value == value:
                    return
            raise AssertionError(
                f"No messages match topic={topic!r} and value={value!r}: "
                f"{self.sent_messages}"
            )

Some real-life examples of test helpers:

.. code:: python

    class EmailHelper(TestCaseHelper):
        """ Captures and makes assertions about emails sent during tests::

                class MyTest(TestCase):
                    mail = EmailHelper()

                    def test_send_email(self):
                        send_email(to="david@wolever.net", ...)
                        self.mail.assert_mail_sent(
                            to="david@wolever.net",
                            body_contains="Hello, David!",
                        )
        """

    class RedisHelper(TestCaseHelper):
        """ Resets the Redis database after each test::

                class MyTest(TestCase):
                    redis = RedisHelper()

                    def test_redis_set(self):
                        cxn = get_redis_connection()
                        cxn.set("some-key", "some-val")
                        self.assertEqual(cxn.get("some-key"), "some-val")

                    def test_redis_get(self):
                        # The key set in the previous test will be cleared
                        self.assertEqual(cxn.get("some-key"), None)
        """

    class MockHelper(TestCaseHelper):
        """ Uses ``mock.patch`` to mock classes, functions, and methods::

                class MyTest(TestCase):
                    mock_foo = MockHelper("my_pkg.foo")

                    def test_foo(self):
                        my_pkg.foo(bar=42)
                        self.mock_foo.assert_called_with(bar=42)
        """

    class FrozenTimeHelper(TestCaseHelper):
        """ Freezes time during tests::

                class MyTest(TestCase):
                    time = FrozenTimeHelper('2020-01-01 12:13:14')

                    def test_create_widget(self):
                        widget1 = create_widget()
                        self.time.set_frozen_time('2020-01-02 12:13:14')
                        widget2 = create_widget()
                        self.assertLess(widget1.created_on, widget2.created_on)
        """
