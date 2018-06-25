[![Build Status](https://travis-ci.org/clinicedc/edc-facility.svg?branch=develop)](https://travis-ci.org/clinicedc/edc-facility) [![Coverage Status](https://coveralls.io/repos/clinicedc/edc-facility/badge.svg?branch=develop&service=github)](https://coveralls.io/github/clinicedc/edc-facility?branch=develop)

# edc-facility

### Customizing appointment scheduling by `Facility`

Appointment scheduling can be customized per `facility` or clinic:

Add each facility to `app_config.facilities` specifying the facility `name`, `days` open and the maximum number of `slots` available per day:

    from edc_facility.apps import AppConfig as EdcAppointmentAppConfig

    class AppConfig(EdcAppointmentAppConfig):

        facilities = {
            'clinic1': Facility(name='clinic', days=[MO, TU, WE, TH, FR], slots=[100, 100, 100, 100, 100])}
            'clinic2': Facility(name='clinic', days=[MO, WE, FR], slots=[30, 30, 30])}

To schedule an appointment that falls on a day that the clinic is open, isn't a holiday and isn't already over-booked:

    from edc_base.utils import get_utcnow
    from .facility import Facility
    
    suggested_datetime = get_utcnow()
    available_datetime = facility.available_datetime(suggested_datetime)


If holidays are entered (in model `Holiday`) and the appointment lands on a holiday, the appointment date is incremented forward to an allowed weekday. Assuming `facility` is configured in `app_config` to only schedule appointments on [TU, TH]:

    from datetime import datetime
    from dateutil.relativedelta import TU, TH
    from django.conf import settings
    from django.utils import timezone

    from .facility import Facility
    from .models import Holiday
    
    Holiday.objects.create(
        name='Id-ul-Adha (Feast of the Sacrifice)',
        date=date(2015, 9, 24)
    )
    suggested_datetime = timezone.make_aware(datetime(2015, 9, 24), timezone=pytz.utc)  # TH
    available_datetime = facility.available_datetime(suggested_datetime)
    print(available_datetime)  # 2015-09-29 00:00:00, TU

The maximum number of possible scheduling slots per day is configured in `app_config`. As with the holiday example above, the appointment date will be incremented forward to a day with an available slot.

