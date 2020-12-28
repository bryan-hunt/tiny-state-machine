# tiny-state-machine
A simple (tiny) state machine in a single header

## Background

I originally wrote this right around the time of graduating from my university
and have used it in various slightly modified forms throughout my career. Based
on a few requests to license and publish it I have placed here under the MIT
license which I think is most suitable for it. 

## Usage

The usage of this is made as simple as possible with little overhead. To that
end the prototype for state functions return no variables and only accept a 
pointer to a context structure. As long as the context is a structure with the
tiny state context structure at the start of it then it can be made to hold any
required information for the state machine. Another requirement placed on the 
application is that each state function must handle the state transition itself.

### Include the header

Make sure to only include the header into module that will be using it
otherwise you'll end up with multiple copies of the same functions in your
resulting application (which may or may not trigger multiple definition
warnings)

```
#include "tiny_state_machine.h"
```

### Define a custom context (recommended)

Rather that always work out of globals you can create a custom context
definition that will hold everything that is required by the state machine

```
typedef struct _state_context {
    tiny_state_ctx      state;      /**< Must be the first element */
    int                 var1;
} state_context, * state_context_ptr;

```

### Defined the states

```
/* Define the client states */
enum {
    STATE_INIT = 0,    /**< Should always start with 0 */
    STATE_CONNECT,
    STATE_RUN,
    STATE_ERROR        /**< Error states can be anywhere but are recommended at the end */
} STATES_DEFS;


static void state_init(void * pCtx)
{
    /* Perform Actions */
    state_context_ptr pState = (state_context_ptr)pCtx;

    if (pState->var1 == 0)
    {
        pState->var1 = 10;
    }

    /* Update State */
    tiny_state_update(ctx, STATE_CONNECT);
}

static void state_connect(void * pCtx)
{
    /* Perform Actions */
    state_context_ptr pState = (state_context_ptr)pCtx;

    if (pState->var1 == 10)
    {
        pState->var1 = 20;

        /* Update State */
        tiny_state_update(ctx, STATE_RUN);
    }
    else
    {
        /* Update State */
        tiny_state_update(ctx, STATE_ERROR);
    }
}

static void state_run(void * pCtx)
{
    /* Perform Actions */
}

static void state_error(void * pCtx)
{
    /* Perform some error handle or stay in state */
}

/* Define the state machine */
tiny_state_def g_states[] =
{
    TINY_STATE_DEF(STATE_INIT,       &state_init),
    TINY_STATE_DEF(STATE_CONNECT,    &state_connect),
    TINY_STATE_DEF(STATE_RUN,        &state_run),
    TINY_STATE_DEF(STATE_ERROR,      &state_error)
};

```

### Add the init and state driver to a running loop or RTOS task

```
state_context   g_state_context;

void main(void)
{
    static int one_time;

    if (!one_time)
    {
        tiny_state_init(&g_state_context, g_states, sizeof(g_states)/sizeof(g_states[0]), STATE_INIT);
        one_time = 1;
    }
    
    for(;;)
    {
        /* Run the state machine*/
        tiny_state_driver(&g_state_context);
    }
}

```

