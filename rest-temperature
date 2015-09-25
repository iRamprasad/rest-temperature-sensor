#include "contiki.h"

#if PLATFORM_HAS_TEMPERATURE

#include <string.h>
#include <stdio.h>
#include "rest-engine.h"
#include "dev/temperature-sensor.h"
#include "contiki.h"
#include "httpd-simple.h"
#include "webserver-nogui.h"
#include "dev/cc2420.h"

#define HISTORY 16
#define REST_RES_TEMPERATURE 1

float floor(float x){ 
  if(x>=0.0f) return (float) ((int)x);
  else        return (float) ((int)x-1);
}


static void res_get_handler(void *request, void *response, uint8_t *buffer, uint16_t preferred_size, int32_t *offset);

/* A simple getter example. Returns the reading from temperature sensor with a simple etag */

RESOURCE(res_temperature, "title=\"Temperature status\";rt=\"temperature\"", res_get_handler, NULL, NULL, NULL);

static void
res_get_handler(void *request, void *response, uint8_t *buffer, uint16_t preferred_size, int32_t *offset)
{
  int temperature = temperature_sensor.value(0);

  unsigned int accept = -1;
  REST.get_header_accept(request, &accept);

  if(accept == -1 || accept == REST.type.TEXT_PLAIN) {
    REST.set_header_content_type(response, REST.type.TEXT_PLAIN);
    snprintf((char *)buffer, REST_MAX_CHUNK_SIZE, "%d", battery);

    REST.set_response_payload(response, (uint8_t *)buffer, strlen((char *)buffer));
  } else if(accept == REST.type.APPLICATION_JSON) {
    REST.set_header_content_type(response, REST.type.APPLICATION_JSON);
    snprintf((char *)buffer, REST_MAX_CHUNK_SIZE, "{'temperature':%d}", battery);

    REST.set_response_payload(response, buffer, strlen((char *)buffer));
  } else {
    REST.set_response_status(response, REST.status.NOT_ACCEPTABLE);
    const char *msg = "Supporting content-types text/plain and application/json";
    REST.set_response_payload(response, msg, strlen(msg));
  }
}
#endif /* PLATFORM_HAS_TEMPERATURE */

/***********************************************************************************************************************/
/***********************************************************************************************************************/

PROCESS(web_sense_process, "Sense Web Demo");

AUTOSTART_PROCESSES(&web_sense_process);

#define HISTORY 16
static int temperature[HISTORY];
static int sensors_pos;

/*---------------------------------------------------------------------------*/
static int
get_temp(void)
{
  return temperature_sensor.value(0);
}

static float get_mytemp(void)
{ 
return (float) (((get_temp()*2.500)/4096)-0.986)*282;
}

/*---------------------------------------------------------------------------*/
/*---------------------------------------------------------------------------*/
PROCESS_THREAD(web_sense_process, ev, data)
{
  static struct etimer timer;

  PROCESS_BEGIN();
  cc2420_set_txpower(31);

  sensors_pos = 0;
  process_start(&webserver_nogui_process, NULL);

  etimer_set(&timer, CLOCK_SECOND * 2);
  SENSORS_ACTIVATE(temperature_sensor);

  while(1) {
    PROCESS_WAIT_EVENT_UNTIL(etimer_expired(&timer));
    etimer_reset(&timer);

    temperature[sensors_pos] = get_mytemp();
    sensors_pos = (sensors_pos + 1) % HISTORY;
  }

  PROCESS_END();
}
/*---------------------------------------------------------------------------*/
