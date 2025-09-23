# WireScript

This file defines the WireScript Language. 
Rough Idea made by AI. 

```wirescript
// Define the electrical service coming into the building
service {
    voltage: 240V;
    type: "residential";
    amperage: 200A;
}

// Define the main electrical panel
panel "Main Panel" {
    amperage: 200A;
    slots: 40;

    // A 40A double-pole breaker for the Kitchen Subpanel
    breaker 40A, double_pole, label: "Kitchen Subpanel";
    
    // An EV charger with a dedicated 50A breaker
    breaker 50A, label: "EV Charger";
}

// Define the kitchen area, which has its own subpanel
area "Kitchen" {
    // A subpanel powered by the main panel
    subpanel "Kitchen Subpanel" {
        amperage: 125A;
        source: "Main Panel";
        slots: 24;

        // General purpose outlet circuit
        circuit "Countertop 1" {
            breaker 20A;
            run #12-2, length: 25ft;
            outlet "Countertop Outlet 1", gfci: true;
            outlet "Countertop Outlet 2";
        }

        // Lighting circuit
        circuit "Recessed Lights" {
            breaker 15A;
            run #14-3, length: 40ft;
            switch "Kitchen Switch", controls: "Light Group A";
            light "Light Group A", type: "recessed", count: 4;
        }

        // Dishwasher circuit
        circuit "Dishwasher" {
            breaker 15A;
            run #14-2, length: 15ft;
            appliance "Dishwasher", load: 12A;
        }
    }
}

// Define the living room area
area "Living Room" {
    // A standard outlet circuit
    circuit "Outlets" {
        breaker 15A;
        run #14-2;
        outlet "East Wall";
        outlet "South Wall";
    }

    // A ceiling fan and light fixture
    circuit "Fan & Light" {
        breaker 15A;
        run #14-3;
        switch "Fan Switch", controls: "Fan";
        switch "Light Switch", controls: "Fan Light";
        fixture "Fan", type: "ceiling_fan";
        fixture "Fan Light", type: "light_kit";
    }
}

// Describe a three-way switch configuration in a hallway
area "Hallway" {
    circuit "Hallway Lights" {
        breaker 15A;
        run #14-3; // Needs 3-conductor wire for 3-way switch
        
        // Define the two switches
        switch "Hallway 3-Way North", type: "3-way", controls: "Hallway Lights";
        switch "Hallway 3-Way South", type: "3-way", controls: "Hallway Lights";
        
        // Define the light fixture controlled by both switches
        fixture "Hallway Light", type: "surface_mount";
    }
}
```