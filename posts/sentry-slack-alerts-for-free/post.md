# Python: Get Sentry alerts in Slack for free using before_send

Sentry Slack alerts is a premium feature.

If you are using the free version of Sentry and there are 
not too many errors created every day, you can use 
Sentry's [before_send](https://docs.sentry.io/platforms/python/configuration/filtering/#example-before_send) feature to send alerts to Slack 
directly from your codebase. 

Only issue is that this 
will not perform deduplication so you will get too many messages 
and/or Slack will ratelimit them. But if the exceptions
are rare in your codebase (like how I keep my codebases!), this will be useful.

I still check Sentry occasionally but this covers most of the cases for my use case.

Initialization:

````python

sentry_sdk.init(
    dsn=os.environ["SENTRY_DSN"],
    environment=ENVIRONMENT,
    before_send=sentry_before_send,
)

````

Function:
````python
def sentry_before_send(event, hint):
    """
    Called by Sentry right before an event is sent.
    We tap into it to push a concise notification to Slack.

    Parameters
    ----------
    event : dict
        The Sentry event payload.
    hint : dict
        Extra data such as ``exc_info`` or ``log_record``.

    Returns
    -------
    dict
        The (unmodified) event so Sentry can proceed as usual.
    """
    try:
        from utils import let_slack_know

        transaction = event.get("transaction")

        if hint.get("exc_info"):
            exc_type, exc_value, _ = hint["exc_info"]
            message_text = f"{exc_type.__name__}: {exc_value}"
        elif hint.get("log_record"):
            message_text = f"Log: {hint['log_record'].getMessage()}"
        else:
            message_text = event.get("message") or "Unknown Sentry event"
        sentry_prefix = ":small_red_triangle:"

        if transaction:
            let_slack_know(f"{sentry_prefix} `{transaction}`\n```{message_text}```")
        else:
            let_slack_know(f"{sentry_prefix} ```{message_text}```")

    except Exception as e:  # Fail-safe: never block Sentry
        # if Slack integration is not working, try to print the error
        # and let Sentry proceed.
        try:
            print(f"Failed to send Sentry event to Slack: {e}")
        except Exception:
            pass

    # Always return the event so Sentry still captures it
    return event
